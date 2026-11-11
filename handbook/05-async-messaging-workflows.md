# 05 — Async Processing, Queues, Events & Workflows

Moving work off the request path is one of the highest-leverage decisions for latency and reliability — and one of the easiest ways to add accidental complexity. Use async processing where it solves a real problem, and design it for duplicates, failures, and replays.

**Applies when:** background jobs, scheduled tasks, emails/notifications, file processing, third-party calls, fan-out, long-running or multi-step processes, event-driven integration.

## Key questions
- Does this work need to finish before responding to the user? If not, why is it synchronous?
- What delivery guarantee is needed (at-most-once, at-least-once, effectively-once)?
- Does ordering matter? Globally, per key, or not at all?
- What happens on failure — retry, compensate, alert a human?
- What's the volume, burstiness, and acceptable lag?
- Who consumes this message/event, and how will its schema evolve?

## 1. When to go async [ALL]
Good candidates:
- Slow operations (> a few hundred ms): reports, media processing, LLM calls, bulk imports/exports.
- Calls to unreliable/rate-limited third parties.
- Side effects not needed for the response: emails, notifications, analytics, webhooks, search indexing, cache warming.
- Load leveling: absorbing bursts that exceed downstream capacity.
- Fan-out to many consumers.
- Scheduled/periodic work.

Keep synchronous when:
- The user needs the result immediately and it's fast and reliable.
- Strong consistency with the response is required.
- The added complexity outweighs the benefit at current scale.

UX implications: async work needs status (pending/processing/done/failed), progress, notification on completion, and a way to retry.

## 2. Mechanism options

| Option | Fits | Trade-offs |
|---|---|---|
| **In-process background task** | Trivial, loss-tolerant work. | Lost on crash/deploy; no retries. Rarely acceptable for anything that matters. |
| **DB-backed job queue** (e.g. Postgres `SKIP LOCKED`-based libraries, Solid Queue, Oban, River, graphile-worker) | Low–moderate volume; transactional enqueue with business data; no new infra. | DB load at high volume; polling latency. |
| **Redis-backed queue** (BullMQ, Sidekiq, Celery+Redis, RQ) | Moderate–high volume, low latency, rich features (delays, priorities, rate limits). | Redis durability config; separate infra; not transactional with DB. |
| **Managed queue** (SQS, Cloud Tasks, Azure Service Bus, Pub/Sub) | Serverless/cloud-native, minimal ops, high scale. | Vendor coupling; visibility timeouts; limited ordering (FIFO variants). |
| **Message broker** (RabbitMQ, NATS, ActiveMQ) | Routing patterns, work queues, pub/sub, request/reply. | Ops burden; cluster management. |
| **Log/stream** (Kafka, Redpanda, Kinesis, Pulsar, NATS JetStream) | High-throughput event streams, replay, multiple independent consumers, ordering per partition, event sourcing, CDC. | Significant ops/conceptual complexity; partition planning; consumer lag management. |
| **Workflow engine** (Temporal, AWS Step Functions, Azure Durable Functions, Inngest, Restate, Airflow/Dagster for data) | Long-running, multi-step processes with retries, timers, human steps, compensation. | New programming model; determinism constraints; infra. |
| **Cron / scheduler** (cron, k8s CronJob, EventBridge Scheduler, job-library schedulers) | Periodic tasks. | Overlap/duplicate runs, missed runs on downtime, timezone/DST issues. |

Considerations:
- Prefer the mechanism already in the stack. A DB-backed queue often avoids a whole new system early on.
- Queue (work distribution, each message processed once by one worker) ≠ stream (durable ordered log, many consumer groups, replay). Choose by semantics, not brand.

## 3. Delivery semantics & idempotency [ALL]
- Assume **at-least-once** delivery; duplicates **will** happen (retries, redeliveries, visibility timeouts, rebalancing, client retries).
- Make every consumer **idempotent**:
  - Natural idempotency (set state to X rather than increment).
  - Deduplication table keyed by message/event ID (with TTL), checked in the same transaction as the effect.
  - Unique constraints on business keys.
  - Idempotency keys passed to downstream APIs (payments etc.).
- "Exactly-once" = at-least-once delivery + idempotent processing. Stream-processor "exactly-once" features only cover their own boundary.

## 4. Reliable publishing: dual-write problem [PROD]
Writing to the DB and publishing a message are two systems — one can succeed while the other fails.
- **Transactional outbox**: write the event to an `outbox` table in the same DB transaction as the state change; a relay publishes and marks it sent (at-least-once).
- **Change Data Capture (CDC)** (e.g. Debezium, logical replication) from the DB log.
- **DB-backed job queue** enqueued in the same transaction.
- Never: "commit, then publish, and hope".

## 5. Retries, failures & dead letters [MVP]
- Retry transient failures with **exponential backoff + jitter**; cap attempts and total time.
- Classify errors: transient (retry) vs permanent (don't retry; fail fast to DLQ / mark failed).
- **Dead-letter queue** (or failed-jobs table) with: payload, error, attempt history; alerting on growth; tooling to inspect, fix, and **redrive**.
- Poison messages must not block a queue or partition.
- Timeouts per job; visibility timeout > max processing time (or heartbeat/extend).
- Graceful shutdown: stop taking new work, finish or release in-flight jobs on deploy/scale-down.
- Jobs should be **resumable/checkpointed** if long-running; or split into smaller jobs.

## 6. Ordering [MVP]
- Most work doesn't need ordering — don't pay for it unless required.
- If needed, scope it: per entity/key (partition key, FIFO message group), not global.
- Handle out-of-order arrival defensively: version numbers / timestamps in events; ignore stale updates.
- Ordering + parallelism trade-off: ordered keys serialize processing.

## 7. Concurrency, throughput & backpressure [PROD]
- Worker concurrency sized to downstream capacity (DB connections, API rate limits), not just CPU.
- Rate-limit jobs that call rate-limited APIs (token bucket per provider/tenant).
- **Fairness / noisy neighbours**: per-tenant queues, weighted fair scheduling, or concurrency caps so one tenant's bulk import doesn't starve others [SCALE].
- Priorities: separate queues for latency-sensitive vs bulk work.
- Autoscale workers on queue depth/age, not CPU alone [SCALE].
- Bound queue size or apply admission control when consumers can't keep up.

## 8. Event design [PROD]
- Name events in past tense, domain language: `invoice.paid`, `user.email_changed`.
- Envelope: `id`, `type`, `version`/schema, `occurred_at`, `source`, `subject`/entity id, `tenant_id`, `correlation_id`/`causation_id`, `data`. Consider the **CloudEvents** spec.
- Schemas registered and versioned (JSON Schema/Avro/Protobuf + schema registry at scale); compatibility rules (backward/forward).
- Events are a public contract between modules — evolve like APIs ([04](04-apis-and-integrations.md)).
- Thin vs fat events (see [04](04-apis-and-integrations.md)); avoid leaking PII into events/logs unnecessarily.
- Event retention and replay policy; consumers must handle replays idempotently.
- Domain events (inside a bounded context) vs integration events (across contexts) — don't expose internal domain events as cross-service contracts by default.

## 9. Multi-step workflows & distributed transactions [PROD]
- **Saga** pattern: sequence of local transactions with **compensating actions** on failure.
  - Choreography (services react to events) — decoupled but hard to follow beyond a few steps.
  - Orchestration (a coordinator drives steps) — explicit, easier to monitor and change.
- Model workflow state explicitly (state machine) and persist it.
- Timeouts and escalation for steps waiting on external parties/humans.
- Consider a durable workflow engine when flows have many steps, long waits, or complex retry/compensation logic.

## 10. Scheduled jobs [MVP]
- Prevent overlapping runs (locks/leases) and duplicate runs across multiple instances (leader election or single scheduler).
- Define behaviour on missed runs (catch up vs skip).
- Use UTC; be careful with DST if business-local schedules are needed.
- Idempotent and resumable; time-bounded; monitored (alert when a job **didn't** run — dead man's switch / heartbeat monitor).

## 11. Observability for async [PROD]
- Metrics: queue depth, **age of oldest message**, processing rate, latency, success/failure/retry counts, DLQ size, consumer lag (streams).
- Propagate trace context through messages (trace/correlation IDs in headers).
- Job dashboard for ops: search jobs by entity, see attempts/errors, retry/cancel.
- Alerts on: oldest message age > SLO, DLQ growth, consumers down, scheduled job missed.

## 12. Testing async [MVP]
- Unit-test handlers as pure functions where possible.
- Integration tests with the real queue (containers) for enqueue/consume wiring.
- Tests for duplicate delivery, out-of-order delivery, failure + retry, poison messages.
- Test mode that runs jobs inline/synchronously for E2E determinism — but also test the real async path.

## Red flags
- Fire-and-forget with no retries or failure visibility for important work.
- Non-idempotent consumers.
- Publishing events after commit without outbox/CDC for important state changes.
- Global ordering requirements nobody can justify.
- Kafka introduced for a few hundred messages a day.
- Unbounded retries / no DLQ; DLQ with no alert or redrive tooling.
- Cron jobs running on every app instance.
- External API calls inside DB transactions.
- No way for users or support to see the status of async work.

## Decisions to record
Queue/stream technology · delivery guarantees · idempotency strategy · retry policy & DLQ handling · ordering requirements · event schema & versioning · workflow orchestration approach · scheduler approach.

## Evidence of done
- Consumers proven idempotent by tests (duplicate delivery test).
- Retry/backoff/DLQ configured and alerting.
- Outbox/CDC used for events tied to state changes (or explicit rationale).
- Queue metrics and alerts exist.
- Runbook for redriving DLQ and handling backlog.

## Related
[02](02-architecture.md) · [04](04-apis-and-integrations.md) · [06](06-reliability.md) · [13](13-observability.md)

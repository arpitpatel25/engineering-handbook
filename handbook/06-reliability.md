# 06 — Reliability & Resilience

Reliability is the system doing what users expect, consistently, even when parts of it fail. It is designed in, not bolted on — through timeouts, isolation, redundancy, graceful degradation, and fast recovery.

**Applies when:** anything with real users; any external dependency; any change to critical paths, deployment, or infrastructure.

## Key questions
- What are the critical user journeys, and what reliability does each need?
- What happens to each journey when each dependency is slow, down, or returns errors?
- How do we detect failure, and how fast do we recover?
- What's the blast radius of a bad deploy, bad config, bad tenant, or bad node?
- Are we spending reliability effort where users feel it?

## 1. Targets: SLIs, SLOs, error budgets [PROD]
- **SLI** (indicator): measured from the user's perspective — e.g. % of requests successful and < 300 ms; % of jobs completed within 5 min.
- **SLO** (objective): target for the SLI over a window — e.g. 99.9% over 28 days.
- **Error budget**: 100% − SLO. Spend it on velocity; when exhausted, prioritize reliability work.
- **SLA** (agreement): contractual, with penalties — set looser than the SLO [ENT].
- Different journeys deserve different SLOs (checkout ≠ avatar upload).
- Don't target 100%; each additional 9 costs roughly an order of magnitude more.

## 2. Timeouts [ALL]
- **Every** network call has a timeout: HTTP clients, DB queries, cache, queue, DNS, SDKs (many default to infinite).
- Timeouts at each layer smaller than the caller's (deadline propagation): if the edge allows 10s, inner calls must finish well within it.
- Separate connect timeout vs read timeout.
- Server-side request timeouts and DB statement timeouts.

## 3. Retries [MVP]
- Retry only **transient** errors (timeouts, 5xx, 429, connection resets) and only **idempotent** operations (or with idempotency keys).
- Exponential backoff with **jitter**; cap attempts; respect `Retry-After`.
- Retry at **one** layer — retries at every layer multiply load (retry storms).
- Retry budgets (e.g. retries ≤ 10% of requests) [SCALE].

## 4. Isolation & containment [PROD]
- **Circuit breakers**: stop calling a failing dependency; fail fast; probe for recovery.
- **Bulkheads**: separate pools (threads, connections, workers, queues) per dependency or workload so one can't exhaust shared resources.
- **Rate limiting** inbound (per user/tenant/IP/key) and outbound (per provider).
- **Load shedding**: reject low-priority work early (with 503/429) under overload instead of degrading everything.
- **Backpressure**: bounded queues/buffers; propagate "slow down" upstream.
- **Blast radius reduction**: per-tenant limits, cell-based architecture, staged rollouts, regional isolation [SCALE].
- **Noisy neighbour protection** in multi-tenant systems.

## 5. Graceful degradation [PROD]
- Define fallbacks per dependency: cached/stale data, default values, reduced functionality, queue for later, hide the feature.
- Non-critical features must not take down critical ones (e.g. recommendations failing shouldn't fail checkout).
- Feature flags / kill switches for risky or expensive features.
- Static fallback pages / maintenance mode.

## 6. Redundancy & high availability [PROD]
- No single points of failure on critical paths: multiple app instances, multi-AZ databases, redundant load balancers.
- Stateless app tier; health-checked; auto-replaced.
- Database HA: replicas with automatic failover; understand failover time and data loss window.
- Multi-region only when SLO, latency, or regulatory needs require it — it's expensive and complex [SCALE/ENT].
- Know your dependencies' SLAs: your availability ≤ product of serial dependencies' availability.

## 7. Health checks & lifecycle [MVP]
- **Liveness** (process is alive; restart if not) vs **readiness** (can serve traffic; remove from LB if not) vs **startup** probes.
- Readiness should not fail just because a non-critical dependency is down (avoid cascading removal of all instances).
- **Graceful shutdown**: on SIGTERM stop accepting new work, drain in-flight requests/jobs, close connections, exit within the grace period.
- Fast, predictable startup; fail fast on invalid config.

## 8. Data integrity [ALL]
- Transactions and constraints for invariants ([03](03-data.md)).
- Idempotency for retried operations ([04](04-apis-and-integrations.md), [05](05-async-messaging-workflows.md)).
- Reconciliation jobs for critical cross-system state (payments vs ledger, billing vs usage) [PROD].
- Checksums / integrity verification for important files and transfers.

## 9. Change safety (most outages are caused by changes) [PROD]
- Small, frequent deploys; automated tests; staged rollout (canary, percentage, ring); automated rollback on SLO regression.
- Config changes treated like code (reviewed, versioned, staged).
- Feature flags to decouple deploy from release ([14](14-delivery-cicd.md)).
- Backward-compatible migrations ([03](03-data.md)).
- Change freeze/approval rules for high-risk windows [ENT].

## 10. Capacity & limits [PROD]
- Know limits: DB connections, file descriptors, memory, disk, API quotas, cloud service limits/quotas.
- Alert before hitting them (disk at 80%, connections at 80%).
- Load test to find breaking points ([07](07-performance-scalability.md)).
- Autoscaling with sensible min/max and scaling on the right signal.
- Resource limits on containers to prevent one process starving others.

## 11. Disaster recovery [PROD/ENT]
- RPO (max data loss) and RTO (max downtime) per system.
- Backups + tested restores ([03](03-data.md)).
- Infrastructure reproducible from code ([15](15-infrastructure-cost.md)).
- DR plan for region/provider loss, account compromise, ransomware, accidental mass deletion.
- DR drills on a schedule [ENT].

## 12. Resilience testing [SCALE]
- Fault injection in tests (timeouts, errors, slow responses from fakes).
- Chaos experiments in staging/production with guardrails (kill instances, add latency, fail dependencies).
- Game days to rehearse incidents.

## Red flags
- HTTP clients with default (infinite) timeouts.
- Retries without backoff/jitter, or at multiple layers.
- A non-critical dependency outage taking down core flows.
- Readiness checks that depend on every downstream service.
- No graceful shutdown → errors on every deploy.
- "Five nines" targets with a single database instance.
- Backups never restored; DR plan never exercised.
- No per-tenant/user limits in a shared system.

## Decisions to record
SLOs per journey · timeout & retry policy · degradation behaviour per dependency · HA topology · RPO/RTO · rollout strategy.

## Evidence of done
- All outbound calls have explicit timeouts (lint or review checked).
- Failure behaviour for each critical dependency is tested.
- Liveness/readiness/graceful shutdown implemented.
- SLOs defined and measured (production stage).
- Restore and failover tested (date recorded).

## Related
[05](05-async-messaging-workflows.md) · [07](07-performance-scalability.md) · [13](13-observability.md) · [14](14-delivery-cicd.md) · [16](16-operations-incidents.md)

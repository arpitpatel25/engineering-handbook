# 02 — Architecture & System Design

Architecture is the set of decisions that are expensive to change. The goal is not to pick the most sophisticated design, but the one that meets the requirements with the least accidental complexity — while keeping future options open.

**Applies when:** starting a system, adding a significant component, splitting/merging services, or when a change crosses many module boundaries.

## Key questions
- What are the main domains/capabilities and their boundaries?
- What must be synchronous vs what can be asynchronous?
- Where does state live, who owns it, and how consistent must it be?
- What are the critical user journeys and their latency/availability needs?
- What fails when each dependency fails?
- Which decisions are irreversible, and which can we defer?
- Who (how many teams) will build and run this?

## 1. Architectural style

| Option | Fits when | Costs |
|---|---|---|
| **Monolith (modular)** | Single team or few teams; early product; domain still evolving. | Needs discipline to keep module boundaries; scales as a unit. |
| **Microservices** | Many teams needing independent deploys; strongly divergent scaling/runtime needs; clear, stable domain boundaries. | Distributed systems complexity: network failures, data consistency, observability, deployment, local dev. |
| **Serverless / FaaS** | Spiky or low traffic, event-driven glue, small team wanting minimal ops. | Cold starts, execution limits, vendor coupling, harder local testing, cost at sustained high load. |
| **Event-driven** | Many consumers of the same facts; decoupled workflows; audit/replay needs. | Eventual consistency, harder debugging, schema evolution of events. |
| **Modular monolith + workers** | Most web products: request path + background processing. | Shared DB coupling if not careful. |
| **Cell-based / shard-per-tenant** | Blast-radius isolation, large enterprise tenants, data residency. | Operational complexity, cross-cell features. |
| **Edge / local-first** | Offline support, global latency, collaborative apps. | Sync/conflict resolution (CRDTs, OT), harder consistency. |

Considerations:
- [ALL] Start from the simplest style that meets requirements; microservices should be a response to an **organizational or scaling** problem you actually have.
- [ALL] Regardless of style, enforce **module boundaries**: each module owns its data, exposes an interface, and doesn't reach into others' internals/tables.
- [SCALE] Extract a service when there's a concrete driver: independent scaling, isolation of failure, different runtime/language, team ownership, compliance isolation.
- [ALL] Conway's Law: the system will mirror the communication structure of the organization — design boundaries accordingly.

## 2. Domain decomposition [MVP]
- Identify **bounded contexts** (areas with their own consistent language and model: Billing, Identity, Catalog, Orders…).
- Map relationships between contexts (who depends on whom; upstream/downstream; shared kernel vs anti-corruption layer).
- Keep a **ubiquitous language** glossary — names in code should match names the business uses.
- Avoid a shared "God model" (e.g. one `User` object that every module mutates).
- Decide where aggregates/transactional boundaries are (what must be updated atomically together).

## 3. Communication patterns
- **Synchronous** (HTTP/gRPC/in-process calls): simple, immediate response; couples availability and latency of caller to callee. Chains of sync calls multiply failure probability and latency.
- **Asynchronous** (queues, streams, events): decouples availability, smooths load, enables retries; adds eventual consistency and operational surface. See [05](05-async-messaging-workflows.md).
- [ALL] Rule of thumb to consider: keep the **user-facing request path short**; move slow, retryable, or non-essential work off it.
- [PROD] Avoid deep synchronous call chains between services; prefer data replication via events or aggregation at the edge (BFF / API gateway) when needed.
- Commands vs events: commands ask something to happen (one handler); events state that something happened (zero-to-many consumers).

## 4. State & data ownership
- [ALL] Every piece of data has exactly **one owner** (module/service) that is the source of truth.
- [MVP] Decide per domain: strong consistency vs eventual consistency (see [01](01-requirements.md) NFRs).
- [SCALE] Shared databases across services couple deploys and schemas — consider per-service schemas/databases once services are separately deployed.
- [PROD] Derived data (caches, search indexes, read models, analytics) must be **rebuildable** from the source of truth.
- [SCALE] CQRS: separate read and write models when their shapes or scaling diverge strongly. Not by default.
- [SCALE] Event sourcing: store state as a sequence of events when audit/history/temporal queries are core. High complexity; rarely needed system-wide.
- Stateless application tier [MVP]: keep session/state in stores, not instance memory, so instances can be replaced and scaled.

## 5. Distributed systems realities [PROD]
- The network is unreliable; latency is not zero; bandwidth is finite; topology changes (the fallacies of distributed computing).
- **CAP / PACELC**: under partition choose consistency or availability; otherwise trade latency vs consistency.
- Exactly-once delivery is generally not achievable end-to-end; design for **at-least-once + idempotency**.
- Clocks drift — don't rely on wall-clock ordering across machines; use sequence numbers, logical clocks, or DB-generated ordering.
- Distributed transactions (2PC) are fragile; prefer sagas/outbox (see [05](05-async-messaging-workflows.md)).
- Partial failure is the normal case: design each interaction's behaviour when the other side is down, slow, or returns garbage.

## 6. Scalability architecture [SCALE]
- Horizontal scaling of stateless tiers behind a load balancer.
- Read scaling: replicas, caches, CDNs, materialized views.
- Write scaling: batching, queues, partitioning/sharding (choose shard key carefully — hard to change).
- Hot-spot avoidance (celebrity users, hot tenants, hot keys).
- Backpressure and load shedding (see [06](06-reliability.md)).
- Multi-region: active-passive vs active-active; data replication and conflict strategy; required only for strict latency, availability, or residency needs.

## 7. Integration architecture
- API gateway / BFF (backend-for-frontend) for aggregating and adapting to clients [SCALE].
- Anti-corruption layer around third-party or legacy systems [MVP] — never let an external model leak through your domain.
- Webhooks in/out, polling, file drops, streaming — see [04](04-apis-and-integrations.md).
- Vendor abstraction: wrap critical vendors (payments, email, LLMs, storage) behind an interface you own — but don't build generic abstractions for vendors you'll never swap.

## 8. Cross-cutting concerns (decide once, apply everywhere)
- Authentication & authorization model ([09](09-identity-access-tenancy.md))
- Tenancy model ([09](09-identity-access-tenancy.md))
- Error model & error propagation ([04](04-apis-and-integrations.md), [11](11-code-design.md))
- Logging/tracing/correlation ([13](13-observability.md))
- Configuration & secrets ([08](08-security.md), [14](14-delivery-cicd.md))
- Time handling (UTC storage, timezone rendering), money handling (integer minor units/decimal, currency), IDs (UUIDv7/ULID/snowflake vs sequential).
- Internationalization ([17](17-frontend-clients.md))
- Feature flags ([14](14-delivery-cicd.md))
- Rate limiting & quotas ([06](06-reliability.md), [18](18-saas-enterprise.md))
- Audit logging ([18](18-saas-enterprise.md))

## 9. Technology selection [ALL]
Evaluate each candidate (language, framework, database, queue, cloud service) against:
- Fit to requirements (not popularity).
- Team familiarity and hiring market.
- Maturity, community, maintenance activity, release cadence, security track record.
- Operational burden (managed vs self-hosted).
- Lock-in and exit cost.
- License compatibility ([20](20-dependencies-supply-chain.md)).
- Cost at expected and 10× scale.
- Observability and tooling support.
- "Innovation tokens": limit the number of novel technologies per project.

## 10. Documenting architecture [MVP]
- **C4 model** diagrams: Context → Containers → Components (code-level rarely needed). Diagrams-as-code (Mermaid, PlantUML, Structurizr) kept in repo.
- Sequence diagrams for critical flows.
- Data flow diagram including trust boundaries (also feeds the threat model).
- ADRs for each significant decision ([templates/adr.md](../templates/adr.md)).
- Keep an "architecture.md" overview short and current; link to details.

## Red flags
- Microservices for a single small team or before product-market fit.
- Services sharing database tables (distributed monolith).
- Synchronous chains of 3+ service hops on the user request path.
- Business logic duplicated across services/clients.
- Introducing a message broker, cache, or new database "for future scale" without a requirement.
- No owner for a piece of data; multiple writers to the same table from different modules.
- Distributed transactions across services.
- "We'll add observability/security later."
- Generic plug-in/abstraction layers with a single implementation.

## Decisions to record
Architectural style · module/service boundaries · sync vs async per flow · data ownership & consistency per domain · database(s) · messaging technology (if any) · ID strategy · tenancy model · deployment topology · region strategy.

## Evidence of done
- Context + container diagram in repo.
- ADRs for each significant decision, including rejected alternatives.
- Failure behaviour documented for each external dependency on critical paths.
- Module boundaries enforced (lint rules, package visibility, architecture tests) where tooling allows.

## Related
[03](03-data.md) · [04](04-apis-and-integrations.md) · [05](05-async-messaging-workflows.md) · [06](06-reliability.md) · [07](07-performance-scalability.md)

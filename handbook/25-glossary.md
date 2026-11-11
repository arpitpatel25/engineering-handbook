# 25 — Glossary of Patterns & Terms

Short definitions for terms used across the handbook. Use these names consistently in specs, ADRs, and code reviews.

| Term | Meaning |
|---|---|
| **ADR** | Architecture Decision Record — short document capturing a decision, its context, alternatives, and consequences. |
| **Aggregate** | DDD cluster of objects treated as one unit for data changes; the transactional consistency boundary. |
| **Anti-corruption layer** | Translation layer that stops an external/legacy model from leaking into your domain. |
| **At-least-once / at-most-once / exactly-once** | Message delivery guarantees. Practical systems use at-least-once + idempotent consumers. |
| **Backpressure** | Mechanism for a slow consumer to signal producers to slow down. |
| **BFF** | Backend-for-Frontend — a backend tailored to a specific client. |
| **Blast radius** | Scope of impact when something fails or a change goes wrong. |
| **Blue-green deployment** | Two identical environments; traffic switches between them for releases and rollback. |
| **BOLA / IDOR** | Broken Object Level Authorization / Insecure Direct Object Reference — accessing another user's object by changing an ID. |
| **Bounded context** | DDD boundary within which a model and language are consistent. |
| **Bulkhead** | Isolating resources per workload/dependency so failures don't spread. |
| **C4 model** | Architecture diagram levels: Context, Containers, Components, Code. |
| **Canary release** | Rolling out to a small subset first and monitoring before expanding. |
| **CAP / PACELC** | Under Partition choose Availability or Consistency; Else trade Latency vs Consistency. |
| **CDC** | Change Data Capture — streaming DB changes from the transaction log. |
| **Cell-based architecture** | Independent copies of the stack, each serving a subset of tenants, to limit blast radius. |
| **Circuit breaker** | Stops calls to a failing dependency for a period, failing fast instead. |
| **CQRS** | Command Query Responsibility Segregation — separate write and read models. |
| **CRDT** | Conflict-free Replicated Data Type — data structures that merge concurrent edits without conflicts. |
| **Dead-letter queue (DLQ)** | Holding area for messages that repeatedly fail processing. |
| **DORA metrics** | Deployment frequency, lead time, change failure rate, time to restore. |
| **Error budget** | Allowed unreliability (100% − SLO) to spend on change. |
| **Event sourcing** | Persisting state as an append-only sequence of events. |
| **Expand/contract migration** | Zero-downtime schema change: add new, migrate, then remove old in later release. |
| **Feature flag** | Runtime switch controlling feature exposure, decoupling deploy from release. |
| **Golden signals** | Latency, traffic, errors, saturation. |
| **Hexagonal architecture** | Ports & adapters — domain at the centre, infrastructure plugged in via interfaces. |
| **Idempotency** | Performing an operation multiple times has the same effect as once. |
| **Idempotency key** | Client-supplied unique key letting the server recognize and dedupe retries. |
| **Jitter** | Randomness added to retry delays/TTLs to avoid synchronized spikes. |
| **Keyset (cursor) pagination** | Paging by "after this key" rather than offset; stable and fast at scale. |
| **Load shedding** | Deliberately rejecting some work under overload to protect the rest. |
| **Modular monolith** | Single deployable with strongly separated internal modules. |
| **Noisy neighbour** | One tenant/workload degrading others on shared resources. |
| **NFR** | Non-functional requirement (quality attribute) — latency, availability, security, etc. |
| **Outbox pattern** | Writing events to a DB table in the same transaction as state changes, then relaying them. |
| **p50 / p95 / p99** | Latency percentiles — the value below which 50/95/99% of requests fall. |
| **Poison message** | A message that always fails processing and can block a queue. |
| **RED method** | Rate, Errors, Duration — service-level metrics. |
| **RLS** | Row-Level Security — DB-enforced per-row access policies. |
| **RPO / RTO** | Recovery Point Objective (max data loss) / Recovery Time Objective (max downtime). |
| **Saga** | Sequence of local transactions with compensating actions instead of a distributed transaction. |
| **SBOM** | Software Bill of Materials — inventory of components in an artifact. |
| **SCIM** | Standard protocol for automated user provisioning from identity providers. |
| **SLI / SLO / SLA** | Indicator (measurement) / Objective (target) / Agreement (contract). |
| **SLSA** | Supply-chain Levels for Software Artifacts — build provenance framework. |
| **SSRF** | Server-Side Request Forgery — tricking the server into requesting internal resources. |
| **Strangler fig** | Incrementally replacing a legacy system by routing functionality to new components. |
| **STRIDE** | Threat categories: Spoofing, Tampering, Repudiation, Information disclosure, DoS, Elevation of privilege. |
| **Tolerant reader** | Consumer that ignores unknown fields, enabling non-breaking evolution. |
| **Transactional outbox** | See outbox pattern. |
| **Trunk-based development** | Short-lived branches merged frequently into main. |
| **Twelve-Factor App** | Methodology for cloud-native apps: config in env, stateless processes, disposability, dev/prod parity, etc. |
| **USE method** | Utilization, Saturation, Errors — resource-level metrics. |
| **UUIDv7 / ULID** | Time-ordered unique identifiers, index-friendly. |
| **Vertical slice** | A change spanning all layers for one narrow feature, delivered end-to-end. |
| **YAGNI** | "You Aren't Gonna Need It" — don't build for speculative requirements. |

# Design Review Checklist

Use before building a significant feature or system. Answer each question or mark N/A with reason.

## Problem
- [ ] Problem, users, goals, and non-goals are explicit
- [ ] NFR targets are numeric (latency, throughput, availability, durability)
- [ ] Back-of-envelope sizing done (rps, storage, fan-out)
- [ ] Riskiest assumptions identified and de-risked

## Design
- [ ] Simplest design that meets requirements; alternatives considered and recorded
- [ ] Fits existing architecture; no new infrastructure without justification
- [ ] Data ownership clear; one writer per entity
- [ ] Consistency model chosen per operation
- [ ] Sync vs async decided per step; user-facing path kept short
- [ ] Contracts (API/event schemas) defined and versionable
- [ ] Failure behaviour defined for each dependency (slow, down, wrong, duplicate)
- [ ] Idempotency and retry semantics defined
- [ ] Scaling bottleneck identified; behaviour at 10× load understood

## Security & privacy
- [ ] Trust boundaries and threats reviewed (STRIDE)
- [ ] AuthN/AuthZ model for new resources, including tenant isolation
- [ ] Personal/sensitive data identified; minimization, retention, deletion covered
- [ ] Abuse cases and rate limits considered

## Operations
- [ ] Observability: what will we measure, what will alert
- [ ] Rollout plan: flags, migrations, backfills, rollback
- [ ] Cost estimate at expected and 10× scale
- [ ] Ownership and on-call implications

## Delivery
- [ ] Broken into vertical slices with acceptance criteria
- [ ] Test strategy per slice
- [ ] Docs/ADRs to be written identified

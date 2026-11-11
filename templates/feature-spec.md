# Feature Spec: <name>

- **Status:** Draft | Approved | Implemented
- **Owner:**
- **Related:** issues, ADRs, designs

## Context & goal
Why this matters; who it's for; measurable success criteria.

## Non-goals
What this explicitly does not do.

## Behaviour
User stories with acceptance criteria:
- **Given** … **When** … **Then** …

## Edge cases & errors
Empty states, limits, concurrency, duplicates, permissions, deleted/suspended entities, timezones, partial failures — and expected behaviour for each.

## Data & contracts
Schema changes (with migration approach), API/event changes (backward compatible?), new config.

## NFRs touched
Latency, load, security/authz, privacy (new personal data?), accessibility, cost.

## Design notes
Approach, sync vs async, dependencies and their failure handling. Alternatives considered (or link ADR).

## Test plan
Unit / integration / E2E / authz denial / failure cases.

## Rollout
Feature flag? Migration/backfill order? Monitoring & alerts? Rollback plan? Docs/changelog?

## Open questions
| Question | Owner | Resolution |
|---|---|---|

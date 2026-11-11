# Code Review Checklist

For humans and agents reviewing a change. Style is the formatter's job — focus here. Not every item applies to every change.

## Intent & scope
- [ ] The change does what the spec/issue asks — no more, no less
- [ ] One concern per PR; refactors separate from behaviour changes
- [ ] Assumptions stated; significant decisions recorded (ADR)

## Correctness
- [ ] Edge cases: empty, null/absent, max sizes, unicode, timezones, concurrency, duplicates
- [ ] Error paths handled meaningfully; nothing swallowed
- [ ] Race conditions / read-modify-write races addressed
- [ ] Transactions scoped correctly; no external calls inside DB transactions
- [ ] Idempotent where retried (jobs, webhooks, payment-like operations)
- [ ] Backward compatibility of APIs, events, configs, and DB schema

## Security & privacy
- [ ] Input validated at boundary; output encoded
- [ ] AuthN required; **object-level authZ** and tenant scoping enforced
- [ ] No secrets, tokens, or PII in code, logs, errors, or URLs
- [ ] No injection vectors (SQL, shell, template, SSRF, path traversal, prompt injection)
- [ ] New dependencies justified, verified to exist, license OK

## Design & maintainability
- [ ] Fits existing architecture and conventions; correct module/layer
- [ ] No unnecessary abstraction, wrapper, config option, or dead code
- [ ] No duplication of existing helpers/utilities
- [ ] Names match domain language; comments explain why
- [ ] Functions/files reasonably sized and cohesive

## Performance & reliability
- [ ] No N+1 queries or unbounded queries/loops/memory
- [ ] Indexes for new query patterns
- [ ] Timeouts/retries on new outbound calls
- [ ] Slow work kept off the request path where appropriate

## Tests
- [ ] New behaviour tested at the right tier, including failure paths
- [ ] Tests would fail without the change
- [ ] No weakened, skipped, or deleted tests without justification
- [ ] Deterministic (no real time/network/random without control)

## Operability
- [ ] Logs/metrics/traces for new flows; alerts + runbook if critical
- [ ] Migrations safe for zero-downtime and rollback
- [ ] Feature flag for risky rollout; flag cleanup planned
- [ ] Config/env changes documented in `.env.example`

## Docs & UX
- [ ] README/AGENTS.md/API docs/changelog updated where relevant
- [ ] UI: loading/empty/error states, accessibility, i18n strings

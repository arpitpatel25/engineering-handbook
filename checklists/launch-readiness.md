# Launch Readiness Checklist

Before exposing a system or major feature to real users. Tailor to stage ([stage-gates.md](stage-gates.md)).

## Functionality
- [ ] Acceptance criteria met; critical journeys pass E2E tests
- [ ] Edge cases and error states handled in UI and API
- [ ] Accessibility checks passed for new UI

## Security
- [ ] Scans clean or triaged (SCA, SAST, secrets, IaC, container)
- [ ] AuthZ/tenant isolation tests pass
- [ ] Rate limits and abuse protections in place
- [ ] Secrets in secret manager; debug modes off; admin surfaces protected
- [ ] Pen test / security review done (if stage/risk requires)

## Reliability & performance
- [ ] Load tested at expected peak (+ headroom); results recorded
- [ ] Timeouts, retries, circuit breakers, degradation paths verified
- [ ] Autoscaling and resource limits configured
- [ ] Backups enabled and restore tested
- [ ] Rollback plan tested; feature flag / kill switch available

## Observability & operations
- [ ] Dashboards for the launch's critical paths
- [ ] Alerts with runbooks; on-call aware and staffed for launch window
- [ ] Error tracking with release tagging
- [ ] Status page / customer comms plan ready
- [ ] Support team briefed; support tooling ready

## Data & privacy
- [ ] Migrations rehearsed on production-like data
- [ ] Privacy policy/terms updated; consent flows working
- [ ] Data retention & deletion paths working
- [ ] Analytics events validated

## Business
- [ ] Billing/entitlements correct for each plan (if applicable)
- [ ] Documentation, changelog, announcement ready
- [ ] Success metrics instrumented and baseline captured
- [ ] Go/no-go owner identified; launch & rollback criteria agreed

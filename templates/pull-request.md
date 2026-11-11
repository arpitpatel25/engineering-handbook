<!-- Copy to .github/pull_request_template.md in your project -->
## What & why
<!-- What does this change and why? Link spec/issue/ADR. -->

## How
<!-- Approach and notable decisions. Alternatives considered? -->

## Risk & rollout
- [ ] Backward compatible (API, events, DB, config)
- [ ] Migration is zero-downtime and rollback-safe (or N/A)
- [ ] Behind feature flag (or N/A)
- [ ] Observability added for new flows (or N/A)

## Testing
<!-- What tests were added/run? Paste relevant output. Manual verification? -->

## Checklist
- [ ] Tests cover new behaviour incl. failure paths
- [ ] AuthZ / tenant isolation considered
- [ ] No secrets/PII in code or logs
- [ ] New dependencies justified
- [ ] Docs / changelog / .env.example updated

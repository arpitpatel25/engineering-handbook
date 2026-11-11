# Stage Gates — Cumulative Checklists by Maturity

Each stage includes everything from the stages before it. Tick what applies; mark the rest **N/A (reason)**. Unchecked + no reason = undecided, which is the thing to avoid.

## Stage: Prototype / Spike
- [ ] Problem and success criteria stated in README (even one paragraph)
- [ ] Secrets outside source control; `.env` gitignored; `.env.example` present
- [ ] Untrusted input validated; parameterized queries
- [ ] Dependencies from trusted sources with lockfile
- [ ] Formatter + linter configured
- [ ] Known shortcuts listed (so they're not mistaken for decisions)

## Stage: MVP (real users / real data)
**Product & design**
- [ ] Project profile written ([templates/project-profile.md](../templates/project-profile.md))
- [ ] Goals, non-goals, NFR sheet with numeric targets
- [ ] Architecture overview + ADRs for key decisions
- [ ] Module boundaries defined

**Data**
- [ ] Schema migrations as code; constraints enforce invariants
- [ ] ID, timestamp (UTC), money conventions decided
- [ ] Tenant/owner scoping on all user-owned data
- [ ] Automated backups enabled
- [ ] Pagination on all list endpoints

**Security & access**
- [ ] Vetted authentication; secure session handling
- [ ] Object-level authorization with denial tests
- [ ] HTTPS, security headers, CSRF/XSS protections
- [ ] Rate limits on auth and abuse-prone endpoints
- [ ] Dependency + secret scanning in CI

**Reliability**
- [ ] Timeouts on every outbound call
- [ ] Slow/side-effect work off the request path with retries
- [ ] Idempotency for payments/external side effects
- [ ] Health endpoint; graceful shutdown

**Quality & delivery**
- [ ] Unit + integration tests for core logic; CI blocks merges on failure
- [ ] One-command local setup
- [ ] Automated or one-click deploy; rollback known

**Observability**
- [ ] Structured logs with request IDs; no secrets/PII in logs
- [ ] Error tracking with release tags
- [ ] Uptime check

**Privacy**
- [ ] Data inventory; privacy policy reflects reality
- [ ] Account deletion & data export path

**Docs**
- [ ] README quickstart verified; AGENTS.md with commands and conventions

## Stage: Production (business-critical)
- [ ] SLOs for critical journeys; alerts on symptoms with runbooks
- [ ] Metrics (RED/USE), tracing, dashboards, deploy markers
- [ ] Threat model; SAST + IaC/container scanning
- [ ] MFA available (required for admins); audit trail of security/admin events
- [ ] Secret manager; rotation process; least-privilege IAM
- [ ] Zero-downtime, backward-compatible migrations
- [ ] Staging environment with parity; immutable artifacts promoted
- [ ] Feature flags/kill switches for risky features
- [ ] Circuit breakers / degradation for critical dependencies
- [ ] DLQ + redrive tooling for async work
- [ ] Load test at expected peak; capacity model recorded
- [ ] Backup **restore tested**; RPO/RTO documented
- [ ] On-call rotation, incident process, postmortem template
- [ ] Infrastructure as code; separate prod account/project
- [ ] Budgets and cost alerts
- [ ] Support tooling instead of ad-hoc production DB access
- [ ] DPAs with processors; retention jobs running

## Stage: Scale
- [ ] Per-tenant limits, fair queuing, noisy-neighbour protection
- [ ] Progressive delivery with automated rollback
- [ ] Caching strategy with invalidation & stampede protection
- [ ] Read replicas/partitioning as needed; capacity planning cadence
- [ ] Tail-sampled tracing; SLO burn-rate alerts; telemetry cost controls
- [ ] Contract tests between services/teams
- [ ] Chaos/game days; DR drills
- [ ] Unit economics (cost per tenant/request)
- [ ] DORA metrics tracked
- [ ] Ownership (CODEOWNERS, service catalog)

## Stage: Enterprise / Regulated
- [ ] SSO (SAML/OIDC), SCIM, enforced MFA, custom roles
- [ ] Customer-facing audit logs with export/streaming
- [ ] Compliance program (SOC 2 / ISO 27001 / HIPAA / etc.) with continuous evidence
- [ ] Pen test reports; vulnerability disclosure; trust center
- [ ] Data residency; BYOK where required
- [ ] Contractual SLAs; public status page
- [ ] SBOM and signed artifacts with provenance
- [ ] Change management & access reviews
- [ ] DR plan tested with documented RTO/RPO achieved
- [ ] Accessibility conformance (VPAT)
- [ ] Self-hosted/single-tenant deployment option (if offered) with upgrade and support policy

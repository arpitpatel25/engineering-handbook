# 00 — Project Profiling & Relevance Triage

Before applying anything else in this handbook, establish **what kind of project this is** and **how much rigor it deserves**. The same feature (e.g. "upload a file") needs very different treatment in a hackathon demo vs. a healthcare platform.

## How to read stage tags

Every item in the handbook carries a tag for the earliest stage where it normally becomes important:

| Tag | Stage | Typical characteristics |
|---|---|---|
| **[ALL]** | Any, including spikes | Cheap hygiene that pays off immediately or prevents irreversible harm. |
| **[MVP]** | Real users or real data | Someone other than the author depends on it; data loss/leaks now hurt. |
| **[PROD]** | Business-critical production | Downtime, data loss, or breaches have financial/reputational cost; on-call exists or should. |
| **[SCALE]** | Significant load/team/data | Multiple teams, high traffic, large datasets, cost becomes material. |
| **[ENT]** | Enterprise / regulated | Selling to enterprises, audits, compliance regimes, contractual SLAs, procurement reviews. |

A project's stage is the **highest** stage it currently is or will be within the planning horizon (~6 months). Items one stage above are worth considering **only** if they are cheap now and expensive to retrofit.

### Cheap-now / expensive-later (consider at every stage)

These are the classic "retrofit is painful" items — consider them early even for MVPs:

- Tenant/owner identifier on every row in multi-user/multi-tenant systems.
- Stable, opaque public IDs (not sequential integers exposed externally) where enumeration matters.
- Timestamps in UTC with timezone-aware types; `created_at` / `updated_at` on persistent records.
- Idempotency for anything involving money, external side-effects, or retried delivery.
- API versioning scheme for public APIs.
- Structured logging with a request/correlation ID.
- Schema migrations as code from day one.
- Secrets outside source control from commit one.
- Separation of config from code (environment-based).
- A license file before anyone else touches the code (if public).
- Internationalization-ready string handling (if any chance of multiple locales).
- Soft vs hard delete semantics and data retention intent.
- Clear module boundaries inside a monolith.

## Profile dimensions

### 1. Maturity stage
`spike` · `prototype` · `MVP` · `production` · `scale` · `enterprise/regulated`

### 2. Project type(s) — can be several
- Web service / API backend
- Web application (SSR/SPA/static)
- Multi-tenant SaaS
- Library / SDK / framework
- CLI / developer tool
- Mobile app (native / cross-platform)
- Desktop app
- Data pipeline / ETL / analytics
- ML system (training / serving)
- LLM / agentic application
- Embedded / IoT / firmware
- Game
- Internal tool / back-office
- Infrastructure / platform tooling
- Browser extension / plugin

See [24 — Project-type addenda](24-project-type-addenda.md) for type-specific concerns.

### 3. Risk profile — worst realistic outcome of a defect
1. Cosmetic / annoyance
2. Lost productivity / bad UX
3. Data loss or corruption (recoverable?)
4. Data exposure (personal / confidential)
5. Financial loss (yours or customers')
6. Legal / regulatory breach
7. Physical safety

Higher risk → stronger testing, review, access control, audit, and change management *in the risky area* (not necessarily everywhere).

### 4. Audience & users
- Internal vs external; consumers vs businesses vs developers.
- Expected user count (now / 12 months), geographic spread, devices, connectivity.
- Accessibility obligations.
- Languages/locales.

### 5. Load & data shape (order of magnitude is enough)
- Requests/sec (avg / peak), read:write ratio, burstiness.
- Data volume now / in 1 year; largest single object; growth rate.
- Latency sensitivity (interactive, near-real-time, batch).

### 6. Constraints
- Team size & skills; who maintains it after launch.
- Budget (infra $/month), deadline.
- Mandated stack, cloud, or vendors; on-prem / air-gapped / data residency needs.
- Compliance regimes (GDPR, HIPAA, PCI DSS, SOC 2, ISO 27001, FedRAMP, etc.).
- Licensing intent (proprietary, open source, open core).

### 7. Lifespan
Throwaway · months · years · decade+. Longer lifespans justify more investment in structure, documentation, and upgradeability.

## Rigor by stage — rough calibration

| Concern | Prototype | MVP | Production | Scale | Enterprise |
|---|---|---|---|---|---|
| Architecture | Whatever's fastest | Modular monolith, clear boundaries | + documented ADRs | + selective extraction, async | + isolation, residency, HA |
| Tests | Smoke / critical logic | Unit + key integration | + E2E on critical paths, CI gate | + perf/load, contract tests | + compliance evidence, DR tests |
| Security | Basics (secrets, input, deps) | + authn/authz, HTTPS, headers | + threat model, scanning, audit trail | + pentest, WAF, bug bounty | + SSO/SCIM, certifications, formal SDLC |
| Observability | Logs | Structured logs + error tracking | + metrics, alerts, dashboards, tracing | + SLOs, error budgets | + audit logs, customer-facing status |
| Delivery | Manual OK | CI + one-click deploy | + staging, migrations, rollback | + progressive delivery, flags | + change management, approvals |
| Ops | None | Backups | + runbooks, on-call, tested restore | + capacity planning, chaos | + DR, RTO/RPO contracts |
| Docs | README | + setup, env vars | + architecture, ADRs, runbooks | + ownership, onboarding | + security/compliance docs |

## Evidence of done
- A written project profile exists (in project AGENTS.md or `docs/project-profile.md`) — see [templates/project-profile.md](../templates/project-profile.md).
- Stage and risk profile are agreed with the user (or marked as assumptions).
- Out-of-scope areas are listed explicitly.

# AGENTS.md — Operating Instructions for AI Agents

You are reading a **generic engineering handbook**. It is not the project you are working on (unless the user is editing this handbook itself). Use it as a decision aid for whatever project you *are* working on.

## Precedence

1. The user's explicit instructions in the current session.
2. The target project's own `AGENTS.md` / `CLAUDE.md` / ADRs / conventions.
3. Existing patterns in the target codebase (consistency beats theoretical improvement).
4. This handbook.

This handbook **never overrides** 1–3. If it conflicts with them, follow them, and — if the conflict looks harmful — say so once, briefly.

## The process

### Step 1 — Profile the project (once per project, or when it changes)

Read [handbook/00-project-profiling.md](handbook/00-project-profiling.md). Determine:

- **Maturity stage**: `spike` · `prototype` · `MVP` · `production` · `scale` · `enterprise / regulated`
- **Project type(s)**: web service, SaaS, library/SDK, CLI, mobile, desktop, data pipeline, ML system, embedded, internal tool, game, …
- **Risk profile**: what is the worst realistic outcome of a defect? (annoyance → data loss → money loss → safety/legal)
- **Constraints**: team size, budget, deadline, existing stack, hosting, compliance obligations.

If the project already has a profile (e.g. a "Project profile" section in its AGENTS.md or `docs/project-profile.md`), use it. If not, infer one and — when it materially affects your decisions — write it down using [templates/project-profile.md](templates/project-profile.md) or state it in your response.

### Step 2 — Triage relevance

Not every area applies. For the task at hand:

- Identify which handbook areas the task touches (a "send email on signup" task touches async processing, reliability, privacy, testing, observability).
- Within each area, each item is tagged with the earliest stage at which it normally matters:
  - **[ALL]** — applies at every stage, including prototypes.
  - **[MVP]** — once real users or real data exist.
  - **[PROD]** — once downtime, data loss, or breaches have real cost.
  - **[SCALE]** — once load, team size, or data volume is significant.
  - **[ENT]** — when selling to / operating inside enterprises or regulated environments.
- Items above the project's stage are **optional** — consider them only if cheap now and expensive later (e.g. tenant IDs on tables, API versioning, idempotency keys).

### Step 3 — Decide

For each relevant consideration:

1. Read the questions. Answer them from the codebase, requirements, or user — **do not invent requirements**.
2. Weigh the listed options and trade-offs against the profile. Prefer the **simplest option that meets the actual requirements**, and prefer what the project already uses.
3. Check the **red flags** list — if your plan matches one, reconsider.
4. If a decision is **hard to reverse, expensive, security-relevant, or user-visible**, and the user hasn't decided it, surface it to the user with a recommendation rather than deciding silently.

### Step 4 — Record

- Significant decisions → an ADR in the target project (`docs/adr/NNNN-title.md`, see [templates/adr.md](templates/adr.md)) or the project's equivalent.
- Deliberate omissions are decisions too ("No caching layer: p95 already 40ms at expected load; revisit at 10× traffic").
- Never silently diverge from an existing ADR; propose superseding it.

### Step 5 — Verify

Each guide ends with **Evidence of done**. Before claiming work complete, check the relevant evidence exists (tests pass, migration is reversible, alert exists, docs updated…). Report honestly what was and wasn't verified.

## Behavioural rules for agents

- **Proportionality over completeness.** Do not gold-plate a prototype with sagas, service meshes, and five-nines SLOs. Do not ship a payments system without idempotency.
- **No unrequested architecture.** Do not introduce new infrastructure (queues, caches, databases, services, frameworks) without a requirement that justifies it and, if non-trivial, the user's agreement.
- **No unrequested dependencies** when a few lines of code or the standard library suffice; evaluate any dependency per [handbook/20](handbook/20-dependencies-supply-chain.md).
- **Consistency first.** Match existing conventions even if this handbook describes a different one; propose changes separately.
- **Small, reviewable changes.** Vertical slices with tests; no sweeping refactors mixed with features.
- **Don't fabricate.** Don't invent SLAs, compliance requirements, load numbers, or user needs. Ask or mark as assumption.
- **Security defaults are not optional** at any stage: no secrets in code, validate untrusted input, parameterized queries, least privilege, dependency hygiene. See [handbook/08](handbook/08-security.md).
- **Leave it better documented than you found it** — but only where documentation earns its keep.

## Quick routing table

| If the task involves… | Read |
|---|---|
| Starting a new project | 00, 01, 02, then `checklists/stage-gates.md` |
| A new feature | `templates/feature-spec.md`, then the areas it touches |
| Schema / data changes | 03 |
| A new endpoint, SDK, webhook, or third-party integration | 04 |
| Background jobs, emails, schedules, events, long-running work | 05 |
| Timeouts, retries, failures, availability | 06 |
| "It's slow" / scaling / caching | 07 |
| Auth, input handling, secrets, files, URLs from users | 08, 09 |
| Personal data, logs with user data, deletion, regulations | 10 |
| Structuring code, refactoring, naming | 11 |
| Writing or fixing tests | 12 |
| Logging, metrics, tracing, alerts | 13 |
| CI, deploys, environments, versioning, releases | 14 |
| Cloud, containers, IaC, cost | 15 |
| Incidents, on-call, runbooks, backups/restore | 16 |
| UI, web/mobile clients, accessibility, i18n | 17 |
| Billing, SSO, audit logs, admin, enterprise asks | 18 |
| LLM calls, RAG, agents, evals | 19 |
| Adding/upgrading packages, licenses, SBOM | 20 |
| README, docs, ADRs | 21 |
| Publishing as open source | 22 |
| Configuring how agents work in a repo | 23 |
| Library, CLI, mobile, data pipeline, ML, embedded, game | 24 |
| Code review | `checklists/code-review.md` |
| Launch / go-live | `checklists/launch-readiness.md` |

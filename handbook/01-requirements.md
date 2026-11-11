# 01 — Problem Framing & Requirements

Most bad architecture comes from unclear requirements, not bad engineering. Before designing, make the problem, constraints, and quality targets explicit.

**Applies when:** starting a project, starting a significant feature, or when design debates keep going in circles.

## Key questions
- What problem are we solving, for whom, and how do we know it's a real problem?
- What does success look like — measurably?
- What are we explicitly **not** doing (non-goals)?
- What are the constraints (time, budget, team, stack, regulation)?
- What are the quality attributes (NFRs) and their targets?
- What assumptions are we making, and which are riskiest?

## 1. Problem & scope [ALL]
- **Problem statement**: one paragraph, user-centred, solution-free.
- **Users & personas**: who, what job they're doing, what they use today.
- **Goals**: measurable outcomes (activation, latency, cost, error rate, time saved).
- **Non-goals**: features, users, platforms, and scales deliberately excluded. Non-goals prevent scope creep and over-engineering.
- **Use cases / user stories** with acceptance criteria (Given/When/Then works well for both humans and agents).
- **Edge cases**: empty states, limits, concurrent edits, partial failures, permission boundaries, deleted/suspended entities, timezones, unicode, very large inputs.
- **Assumptions & open questions** — tracked, owned, and resolved before they become load-bearing.

## 2. Functional requirements [ALL]
- Core flows (happy path) and their failure paths.
- Data that must be captured, derived, retained, deleted.
- Integrations (inbound/outbound), with ownership and SLAs of each.
- Admin/support needs (impersonation, overrides, manual retries, data fixes) — often forgotten until production [MVP].
- Notifications (who, when, channel, opt-out) [MVP].

## 3. Non-functional requirements (quality attributes)

Write targets as numbers where possible. "Fast" is not a requirement; "p95 < 300 ms at 200 rps" is. Use [templates/nfr-sheet.md](../templates/nfr-sheet.md).

| Attribute | Questions / example targets | Stage |
|---|---|---|
| **Latency** | p50/p95/p99 per interaction class (interactive API, page load, background job completion). Never only averages. | MVP |
| **Throughput** | Requests/sec, jobs/min, events/sec — average and peak; burst factor. | MVP |
| **Availability** | Target (99%, 99.9%, 99.99%) per user journey; planned maintenance allowed? 99.9% ≈ 43 min/month downtime. | PROD |
| **Durability** | Which data can never be lost? Acceptable loss window (RPO). | MVP |
| **Recoverability** | Time to restore service (RTO); time to restore data. | PROD |
| **Consistency** | Where is strong consistency required (money, inventory, permissions)? Where is eventual OK (feeds, analytics, search)? Read-your-writes needs? | MVP |
| **Scalability** | Growth expected in 12–24 months; which dimension (users, tenants, data, traffic)? | SCALE |
| **Security** | Data classification, threat actors, authentication strength, audit needs. | ALL |
| **Privacy** | Personal data collected, purpose, retention, residency, deletion. | MVP |
| **Compliance** | Applicable regimes and certifications. | ENT |
| **Usability & accessibility** | WCAG level, supported devices/browsers, offline needs. | MVP |
| **Localization** | Locales, RTL, currencies, timezones. | MVP |
| **Maintainability** | Who maintains it, for how long; onboarding time target. | PROD |
| **Operability** | On-call expectations, self-healing, deploy frequency. | PROD |
| **Portability** | Cloud-agnostic? On-prem / air-gapped deployment? | ENT |
| **Cost** | Budget per month; cost per user/tenant/request target. | SCALE |
| **Observability** | What must be measurable to know it's working? | PROD |
| **Interoperability** | Standards required (OIDC, SAML, SCIM, OpenAPI, CSV export, webhooks). | ENT |

### Back-of-envelope estimation [MVP]
Do quick sizing before choosing technology:
- Daily active users × actions/user/day ÷ 86,400 = avg rps; × peak factor (often 3–10×).
- Records/day × bytes/record × retention days = storage.
- Fan-out: one user action → how many reads/writes/messages downstream?
- Hot set size vs memory available (cache viability).
Order-of-magnitude answers are enough to rule options in or out.

## 4. Constraints & context [ALL]
- Mandated technologies, hosting, vendors, procurement rules.
- Existing systems to integrate with or replace (and migration path).
- Team skills — an unfamiliar stack is a real cost.
- Deadlines and what's negotiable (scope, quality, date).

## 5. Risk analysis [MVP]
- What are the riskiest assumptions (technical, product, market)? De-risk them first with spikes/prototypes.
- Pre-mortem: "It's 6 months later and this failed — why?"
- Irreversible decisions (data model, public API, vendor lock-in, licensing) deserve more scrutiny than reversible ones.

## 6. Specs as the unit of work for agents [ALL]
For AI-assisted development, a feature spec is the single most effective lever on output quality. Use [templates/feature-spec.md](../templates/feature-spec.md):
- Context and goal; non-goals.
- Behaviour with acceptance criteria.
- Edge cases & error handling expectations.
- Data/API changes.
- NFRs touched (latency, security, privacy).
- Test expectations.
- Rollout plan (flag? migration? backfill?).

## Red flags
- Architecture chosen before requirements exist ("we'll use Kafka and microservices").
- No non-goals listed.
- NFRs stated as adjectives ("scalable", "secure", "fast").
- Designing for 100M users with 100 expected.
- Requirements invented by an agent and treated as fact.
- No owner for open questions.

## Decisions to record
- Scope & non-goals; NFR targets; consistency model per domain; supported platforms; compliance scope.

## Evidence of done
- Problem statement, goals, non-goals written.
- NFR sheet with numeric targets (or explicit "not a concern at this stage").
- Acceptance criteria for in-scope stories.
- Assumptions list with owners.

## Related
[00](00-project-profiling.md) · [02](02-architecture.md) · [templates/feature-spec.md](../templates/feature-spec.md) · [templates/nfr-sheet.md](../templates/nfr-sheet.md)

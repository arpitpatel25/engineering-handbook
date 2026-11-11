# 16 — Operations & Incident Management

Operating software is a discipline: knowing who owns what, responding to incidents calmly, learning from them, and continuously reducing toil.

**Applies when:** anything users depend on. Lightweight at MVP, formal at production and beyond.

## Key questions
- Who gets alerted, and do they know what to do?
- How do we communicate during an incident — internally and to customers?
- How do we make sure the same failure doesn't happen twice?
- What routine operational tasks exist, and can they be automated?

## 1. Ownership [MVP]
- Every service, job, data store, and alert has an owning person/team (service catalog: Backstage, OpsLevel, or a simple `OWNERS`/README table) [SCALE for catalog].
- Escalation paths documented.

## 2. On-call [PROD]
- Rotation with primary/secondary; sustainable load (track pages per shift; fix noisy alerts).
- Handoff notes between shifts.
- Tooling for paging, acknowledgement, escalation.
- Access needed during incidents works (break-glass tested) [ENT].
- Compensation/time-off norms for on-call (human sustainability matters for reliability).

## 3. Runbooks [PROD]
- One per alert and per common operational task: what it means, how to diagnose (dashboards, queries), how to mitigate, how to escalate. Template: [templates/runbook.md](../templates/runbook.md).
- Kept next to code/alerts; tested during game days.
- Automate runbook steps over time (runbooks → scripts → self-healing).

## 4. Incident response [PROD]
- **Severity levels** defined by user impact (e.g. SEV1: major outage/data loss/security breach; SEV2: significant degradation; SEV3: minor).
- Roles for significant incidents: **Incident Commander**, communications lead, operations/subject-matter responders, scribe.
- Process: detect → declare → triage → **mitigate first** (rollback, flag off, failover, scale) → resolve root cause later → communicate → close → review.
- Dedicated incident channel; timeline captured as you go.
- Customer communication: status page (Statuspage, Instatus, Better Stack, etc.), templated updates at regular intervals [PROD/ENT].
- Security incidents follow a variant with legal/privacy involvement and notification obligations ([10](10-privacy-compliance.md)).

## 5. Post-incident review [PROD]
- **Blameless** postmortem for significant incidents: summary, impact, timeline, root/contributing causes (5 whys, contributing factors rather than single root cause), what went well, what didn't, action items with owners and dates. Template: [templates/postmortem.md](../templates/postmortem.md).
- Track action items to completion; review trends across incidents.
- Share learnings broadly.

## 6. Backups, restore & DR operations [PROD]
- Scheduled restore tests with recorded results ([03](03-data.md)).
- DR runbooks; periodic drills ([06](06-reliability.md)).

## 7. Routine operations [PROD]
- Dependency/OS/runtime patching cadence; certificate renewal (automated); secret rotation; access reviews [ENT].
- Capacity reviews; cost reviews ([15](15-infrastructure-cost.md)).
- Data maintenance: vacuum/analyze, index maintenance, partition management, retention jobs.
- Upgrade plans for end-of-life runtimes, databases, and frameworks — track EOL dates.

## 8. Toil reduction [SCALE]
- Identify repetitive manual work; automate or eliminate.
- Support tooling: admin console, safe data-fix scripts (reviewed, logged), job retry UI ([18](18-saas-enterprise.md)).
- Self-service for common customer requests.

## 9. Production access & change hygiene [PROD]
- No routine manual changes in production; when necessary, peer-reviewed and logged.
- Database console access restricted, read-only by default, audited.
- Scripts for one-off data fixes go through code review and run via controlled jobs.

## 10. Support integration [MVP]
- Path from customer report → ticket → engineering with request IDs/screenshots/context.
- Support tiers and SLAs [ENT].
- Known issues list; feedback loop into prioritization.

## Red flags
- Alerts with no owner or runbook.
- Same incident recurring; postmortem actions never completed.
- Blame-focused reviews (people hide information next time).
- Hero culture: one person who knows how to fix things.
- Root SSH into production to fix data by hand, unlogged.
- No status page or customer comms plan for a B2B product.

## Decisions to record
Severity definitions · on-call model · status communication approach · postmortem policy.

## Evidence of done
- Every alert links to a runbook.
- Incident process documented and rehearsed (production stage).
- Postmortem actions tracked to closure.
- Restore/DR drills performed on schedule.

## Related
[06](06-reliability.md) · [13](13-observability.md) · [15](15-infrastructure-cost.md) · [templates/runbook.md](../templates/runbook.md) · [templates/postmortem.md](../templates/postmortem.md)

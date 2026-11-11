# 18 — SaaS & Enterprise Product Capabilities

Beyond core features, B2B and enterprise customers evaluate a product on capabilities that let them adopt, govern, integrate, and trust it. Many are cheap to design for early and very expensive to retrofit.

**Applies when:** building multi-customer SaaS, selling to businesses, facing security questionnaires or procurement, or planning self-hosted/on-prem offerings.

## Key questions
- What do customer admins (not end users) need to manage, govern, and audit their usage?
- How do customers get data in and out, and integrate with their other systems?
- How are plans, entitlements, usage, and billing modelled?
- How do support and operations staff help customers safely?
- What will a security/procurement review ask, and do we have answers?

## 1. Accounts, organizations & tenancy [MVP]
- Organization/workspace model, membership, roles, invitations, ownership transfer ([09](09-identity-access-tenancy.md)).
- Multiple orgs per user; org switching.
- Tenant lifecycle: trial → active → past due → suspended → cancelled → deleted, with defined data behaviour at each step.
- Sandbox/test organizations or environments for customers [ENT].

## 2. Enterprise identity [ENT]
- SSO (SAML/OIDC), enforced SSO, domain verification, JIT provisioning.
- SCIM provisioning/deprovisioning; group-to-role mapping.
- Custom roles and fine-grained permissions.
- Session policies (timeout, IP allow-lists, MFA enforcement).
- Managed solutions exist (WorkOS, Auth0/Okta, Frontegg, Stytch, Keycloak) — consider buy vs build.

## 3. Audit logs [PROD → ENT]
- Customer-visible audit trail: who (actor, IP, user agent), did what (action), to what (resource), when, outcome, via which channel (UI/API/SCIM).
- Immutable/append-only, tamper-evident; long retention; searchable/filterable; exportable; streamable to customer SIEM (Splunk, Datadog, S3, webhooks) [ENT].
- Covers admin, security, permission, data export, and configuration changes at minimum.

## 4. Plans, entitlements & billing [MVP]
- Separate **entitlements** (what a tenant may use: features, limits, seats) from **billing** (what they pay). Check entitlements in code via a single service/flag system, not scattered plan-name checks.
- Pricing models: flat, per-seat, usage-based, tiered, hybrid; trials; discounts; annual vs monthly.
- **Usage metering** for usage-based pricing: accurate, idempotent, auditable event capture; aggregation; reconciliation with invoices [PROD].
- Billing provider (Stripe, Paddle, Chargebee, Lago, Orb, Metronome) — let them handle invoicing, tax (VAT/GST/sales tax), dunning, payment methods; your system is the source of truth for entitlements, synced via webhooks ([04](04-apis-and-integrations.md)).
- Proration, upgrades/downgrades, seat changes, cancellations, refunds, credits.
- Invoicing/PO/wire payments for enterprise; multi-currency [ENT].
- Quotas and limits enforced with clear UX and API errors; overage behaviour defined.

## 5. Admin console & settings [MVP]
- Customer admin console: members, roles, security settings, billing, usage, integrations, API keys, audit logs, data export.
- Configurable defaults and policies per org.
- Notifications preferences per user and org.

## 6. Internal back-office / support tooling [PROD]
- Look up tenants/users/entities; view state and history; job/webhook delivery status.
- Safe impersonation ("view as") with consent/notice, scoping, and audit ([09](09-identity-access-tenancy.md)).
- Manual actions (extend trial, apply credit, retry job, reset MFA) with audit trails and permission checks — instead of ad-hoc production SQL.
- Feature flag/entitlement overrides per tenant.

## 7. Data portability & integrations [PROD → ENT]
- Bulk import (CSV, API) with validation, preview, error reporting, idempotency.
- Bulk export (CSV/JSON, full account export) ([10](10-privacy-compliance.md)).
- Public API + webhooks ([04](04-apis-and-integrations.md)); API keys management.
- Native integrations with common tools (Slack, Teams, Google Workspace, Microsoft 365, Salesforce, Jira, Zapier/Make) as needed.
- Embeddability (iframes, SDKs, white-labelling, custom domains) [ENT].

## 8. Trust, security & compliance posture [ENT]
- Security page / trust center; SOC 2 Type II, ISO 27001, pen test summaries; sub-processor list ([10](10-privacy-compliance.md)).
- Ready answers for security questionnaires.
- Data residency options; customer-managed keys (BYOK/HYOK); private networking (PrivateLink).
- Contractual SLAs with measurable uptime and support response times; status page with incident history.
- DPA, MSA, and legal terms aligned with actual practices.

## 9. Reliability & isolation for tenants [SCALE]
- Per-tenant rate limits, quotas, fair queuing; noisy-neighbour protection ([06](06-reliability.md)).
- Dedicated/isolated deployments for large tenants (cell or silo).
- Tenant-level observability and status (which tenants are affected by an incident).

## 10. Deployment models [ENT]
- Multi-tenant cloud (default), single-tenant cloud, customer-cloud (BYOC), on-prem, air-gapped.
- If self-hosted: packaging (Helm/Compose/installer), upgrade path, version support policy, license keys, telemetry opt-in, support bundle generation, documentation of supported infra matrix.

## 11. Customer lifecycle & communication [MVP]
- Onboarding flows and activation tracking; in-app guidance.
- Transactional email (deliverability: SPF, DKIM, DMARC; bounce/complaint handling; templating; localization).
- In-app notifications; announcements/changelog.
- Account deletion and offboarding with data export.
- Support channels and integration with support tools.

## 12. Product analytics & customer health [SCALE]
- Usage per tenant, feature adoption, health scores for customer success; PII-respecting.

## Red flags
- `if plan == "pro"` scattered across the codebase.
- Support staff fixing customer issues via direct production DB access.
- No audit log for admin/security actions.
- Billing state only in the payment provider, not reflected in your system (or vice versa without reconciliation).
- Account deletion that leaves data in third parties.
- Enterprise SSO bolted on with per-tenant special cases.

## Decisions to record
Tenancy & org model · entitlement model · billing provider & sync · audit log scope/retention · deployment models offered · support access model.

## Evidence of done
- Entitlements enforced centrally and tested per plan.
- Audit events emitted for security/admin actions (tested).
- Billing webhooks idempotent with reconciliation.
- Export/deletion flows complete.

## Related
[04](04-apis-and-integrations.md) · [09](09-identity-access-tenancy.md) · [10](10-privacy-compliance.md) · [16](16-operations-incidents.md)

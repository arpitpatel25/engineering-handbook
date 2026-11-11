# 09 — Identity, Access & Multi-Tenancy

Who is this, what may they do, and whose data is it? Getting identity, authorization, and tenant isolation right is foundational — and painful to retrofit.

**Applies when:** any system with users, accounts, API keys, roles, organizations/workspaces, or shared infrastructure across customers.

## Key questions
- Who are the principals (end users, admins, support staff, service accounts, API clients, other services)?
- How do they authenticate, and how strong must it be?
- What is the authorization model, and where is it enforced?
- What is a tenant, and how strictly must tenants be isolated?
- How are identities provisioned, changed, and deprovisioned?

## 1. Authentication [ALL]
- **Build vs buy**: managed identity providers (Auth0/Okta, Clerk, Cognito, Firebase Auth, Supabase Auth, WorkOS, Keycloak, Zitadel, Ory) vs framework libraries. Rolling your own auth protocols is a red flag; using a vetted library in-app is fine.
- Methods: password, passwordless (magic link, OTP), social/OIDC, **passkeys/WebAuthn**, enterprise SSO (SAML/OIDC) [ENT].
- Passwords [MVP]: strong hashing (Argon2id/bcrypt), breach-password checks, length over complexity rules (NIST 800-63B), rate limiting and lockout/backoff, secure reset flow (single-use, expiring tokens, notify on change).
- **MFA** [PROD]: TOTP, WebAuthn; enforceable per org [ENT]; recovery codes; MFA required for admins.
- Email verification and change flows (verify new address, notify old) [MVP].
- Account enumeration resistance [MVP].

## 2. Sessions & tokens [MVP]
- Server-side sessions (opaque cookie) vs stateless tokens (JWT): JWTs are hard to revoke — use short expiry + refresh tokens + revocation strategy if chosen.
- Cookie flags (`Secure`, `HttpOnly`, `SameSite`); session rotation on login/privilege change; idle and absolute timeouts.
- Logout invalidates server-side state; "log out all devices"; session list for users [PROD].
- JWT hygiene: validate signature, `alg` allow-list, `iss`, `aud`, `exp`, `nbf`; key rotation via JWKS.
- OAuth2/OIDC flows: Authorization Code + PKCE for browser/mobile; never implicit flow; client credentials for machine-to-machine.
- Token storage on clients: avoid `localStorage` for long-lived tokens in browsers when XSS is a risk; secure storage on mobile.

## 3. Machine identities [MVP]
- API keys: scoped, hashed at rest, prefixed, shown once, rotatable, expirable, attributable to a user/org, last-used tracked.
- Service-to-service: mTLS, workload identity, signed tokens; no shared static passwords [PROD].
- Personal access tokens vs org-owned service accounts [ENT].

## 4. Authorization [ALL]
- **Enforce server-side, centrally** — one policy layer/middleware/service, not ad-hoc checks sprinkled through code.
- Check at the **object level**: "can this principal perform this action on this resource?" — not just "is logged in" or "has role".
- Models:
  - **RBAC** (roles → permissions): simple, common. Define permissions as fine-grained strings; roles as bundles.
  - **ABAC** (attributes/policies): context-aware rules (owner, department, time, resource state).
  - **ReBAC** (relationship-based, Zanzibar-style: OpenFGA, SpiceDB, Permify): hierarchical sharing (folders, docs, teams) [SCALE].
  - Policy engines (OPA/Rego, Cedar, Casbin, Oso) for externalized, testable policies [SCALE].
- Default deny; explicit grants.
- Authorization decisions also filter **lists/search/exports**, not only single-object fetches.
- Admin & support access: separate privileged roles, scoped, time-bound, audited; impersonation with explicit indicators and audit trail [PROD].
- Custom roles [ENT].
- Authorization unit tests for each policy; integration tests for denial paths.

## 5. Multi-tenancy [MVP when multi-customer]
Define what a **tenant** is (organization, workspace, account, team) and whether users can belong to multiple tenants.

| Isolation model | Isolation | Cost / complexity | Notes |
|---|---|---|---|
| **Shared DB, shared schema, `tenant_id` column** | Logical | Lowest | Most common; requires rigorous scoping; consider DB row-level security as defence in depth. |
| **Schema per tenant** | Stronger logical | Medium | Migrations × N schemas; connection handling. |
| **Database per tenant** | Strong | Higher | Per-tenant backup/restore, residency, noisy-neighbour isolation; ops at scale. |
| **Stack/cell per tenant (silo)** | Strongest | Highest | Enterprise/regulated tenants, dedicated infra, BYO cloud. |
| **Hybrid (pool + silo tiers)** | Mixed | Medium–high | Silo for premium/regulated tenants. |

Considerations:
- Tenant context resolved once per request (subdomain, header, token claim, path) and propagated implicitly (request context) — every query scoped automatically via repository layer, ORM scopes, or RLS.
- Tenant ID on every tenant-owned row, in cache keys, object storage paths, search indexes, queue messages, logs, metrics labels (careful with cardinality), and traces.
- Cross-tenant access tests in CI.
- Per-tenant limits/quotas and fair scheduling ([06](06-reliability.md), [05](05-async-messaging-workflows.md)).
- Per-tenant configuration and feature flags.
- Tenant lifecycle: create, suspend, export, delete (complete and verifiable).
- Data residency per tenant [ENT].
- Per-tenant encryption keys / BYOK [ENT].
- Tenant-aware observability: identify the tenant behind errors/latency [PROD].

## 6. Identity lifecycle [PROD → ENT]
- Invitations (expiring, single-use), joining via verified domain [ENT], ownership transfer, leaving an org.
- Deprovisioning: revoke sessions, tokens, keys on removal.
- **SSO** (SAML 2.0 / OIDC) with JIT provisioning [ENT].
- **SCIM** for automated user/group provisioning and deprovisioning [ENT].
- Group → role mapping from IdP [ENT].
- Domain verification and SSO enforcement [ENT].

## 7. Audit [PROD]
Log identity & access events: login, logout, failed login, MFA changes, password resets, role changes, invitations, API key lifecycle, impersonation, SSO config changes. See [18](18-saas-enterprise.md) for customer-facing audit logs.

## Red flags
- `WHERE id = ?` without tenant/owner scoping.
- Role checks only in UI or route middleware but not on object access.
- Long-lived JWTs with no revocation.
- Shared admin accounts; support staff with unscoped production DB access.
- Tenant ID taken from client-supplied body instead of authenticated context.
- Cache keys or storage paths missing tenant ID.
- Deleting a user without revoking tokens/sessions.

## Decisions to record
Identity provider/approach · session/token design · authorization model & enforcement point · tenancy model · tenant resolution · admin/support access model.

## Evidence of done
- Automated tests proving cross-user and cross-tenant access is denied for each resource type.
- Central authorization layer with tests per policy.
- MFA available (required for admins) at production stage.
- Deprovisioning revokes all credentials.

## Related
[04](04-apis-and-integrations.md) · [08](08-security.md) · [10](10-privacy-compliance.md) · [18](18-saas-enterprise.md)

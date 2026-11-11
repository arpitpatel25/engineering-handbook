# Security Baseline Checklist

The minimum security bar by stage. See [handbook/08-security.md](../handbook/08-security.md) for detail.

## Every project [ALL]
- [ ] No secrets in repo/history; `.env` ignored; secret scanning pre-commit + CI
- [ ] Input validation at all boundaries (schemas)
- [ ] Parameterized queries; no shell/template string injection
- [ ] Framework output encoding; no raw HTML from users without sanitizer
- [ ] Lockfile committed; dependency vulnerability scan
- [ ] Least-privilege credentials for DB, cloud, CI
- [ ] Debug off and generic errors in production

## Real users [MVP]
- [ ] Vetted auth library/provider; strong password hashing; reset flow secure
- [ ] Session cookies `Secure`/`HttpOnly`/`SameSite`; rotation on login
- [ ] Object-level authorization + tenant scoping; denial tests
- [ ] HTTPS + HSTS; CSP and standard security headers; CORS restricted
- [ ] CSRF protection for cookie auth
- [ ] Rate limiting on auth, signup, reset, OTP, expensive endpoints
- [ ] File uploads: size/type limits, safe serving
- [ ] SSRF protections for user-supplied URLs
- [ ] `SECURITY.md` (public repos)

## Production [PROD]
- [ ] Threat model documented
- [ ] MFA available; required for admins and production access
- [ ] Secret manager; rotation; short-lived CI credentials (OIDC)
- [ ] SAST, IaC, and container scanning in CI; CI actions pinned
- [ ] Private networking for data stores; WAF/DDoS protection
- [ ] Security event logging and alerting
- [ ] Incident response plan incl. breach notification
- [ ] Encryption at rest; KMS-managed keys

## Enterprise [ENT]
- [ ] SSO/SCIM; custom roles; audit logs for customers
- [ ] Periodic pen tests; vulnerability disclosure/bug bounty
- [ ] SBOM, signed artifacts, provenance
- [ ] Access reviews; JIT/break-glass production access
- [ ] Compliance framework controls & evidence

# 08 — Security

Security is a property of every layer and every stage. The baseline items below apply even to prototypes, because breaches and leaked secrets are irreversible. Higher-stage items scale with risk.

**Applies when:** always. Especially: handling input, auth, secrets, files, URLs, payments, personal data, infrastructure, dependencies.

## Key questions
- What are we protecting (assets), from whom (threat actors), and what's the impact if compromised?
- Where are the trust boundaries, and what crosses them?
- What's the attack surface (endpoints, inputs, integrations, admin tools, CI/CD, dependencies)?
- How would we know we've been compromised, and how would we respond?

## 1. Baseline — non-negotiable at every stage [ALL]
- No secrets in source control, logs, error messages, client bundles, or URLs. Secret scanning in pre-commit and CI.
- Validate all untrusted input at the boundary (type, length, format, range); allow-list over deny-list.
- Parameterized queries / ORM bindings — never string-concatenated SQL/NoSQL/shell/LDAP.
- Output encoding appropriate to context (HTML, attribute, JS, URL); use framework auto-escaping.
- HTTPS everywhere; HSTS [MVP].
- Authenticate and authorize every non-public operation server-side (client-side checks are UX, not security).
- Least privilege for users, services, DB accounts, cloud IAM, CI tokens.
- Dependencies from trusted sources, pinned via lockfile, scanned for known vulnerabilities ([20](20-dependencies-supply-chain.md)).
- Safe defaults: deny by default, secure cookie flags, debug off in production.
- Don't roll your own cryptography, password hashing, session management, or auth protocols.

## 2. Threat modeling [PROD]
- Draw the data flow with trust boundaries; enumerate threats per element using **STRIDE** (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege) or attack trees.
- Rate by likelihood × impact; record mitigations or accepted risks.
- Revisit on major changes. Template: [templates/threat-model.md](../templates/threat-model.md).

## 3. OWASP Top 10 & API Top 10 coverage [MVP]
- **Broken access control / BOLA / IDOR**: object-level authorization on every access; tenant scoping; tests that user A can't access user B's objects.
- **Cryptographic failures**: TLS in transit; encryption at rest for sensitive data; modern algorithms; no sensitive data in URLs.
- **Injection**: SQL, NoSQL, OS command, template, LDAP, XPath, header injection; also **prompt injection** for LLM features ([19](19-ai-llm-features.md)).
- **Insecure design**: threat modeling, abuse cases, rate limits on sensitive flows.
- **Security misconfiguration**: default credentials, verbose errors, open storage buckets, permissive CORS, unnecessary services/ports.
- **Vulnerable & outdated components**: SCA scanning, update cadence.
- **Identification & authentication failures**: see [09](09-identity-access-tenancy.md).
- **Software & data integrity failures**: signed artifacts, verified updates, safe deserialization, CI/CD integrity.
- **Logging & monitoring failures**: security-relevant events logged and alerted ([13](13-observability.md)).
- **SSRF**: validate/allow-list outbound URLs from user input; block private/link-local/metadata IPs; re-validate after DNS resolution and redirects; use egress proxies where possible.
- **Mass assignment**, **excessive data exposure**, **unrestricted resource consumption** (API Top 10).

## 4. Web-specific protections [MVP]
- **XSS**: auto-escaping templates/frameworks; avoid raw HTML injection APIs; sanitize rich text with a vetted sanitizer; **Content Security Policy**.
- **CSRF**: SameSite cookies + CSRF tokens for cookie-authenticated state-changing requests.
- **Clickjacking**: `frame-ancestors` CSP / `X-Frame-Options`.
- Security headers: `Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `Permissions-Policy`.
- Cookies: `Secure`, `HttpOnly`, `SameSite`, scoped domain/path, `__Host-` prefix where possible.
- CORS: explicit origins; no `*` with credentials.
- Open redirects: allow-list redirect targets.
- Subresource Integrity for third-party scripts; minimize third-party scripts.

## 5. Secrets management [ALL → PROD]
- [ALL] Environment variables or secret files outside the repo; `.env` in `.gitignore`; `.env.example` with placeholders.
- [PROD] Central secret manager (Vault, cloud secret managers, Doppler, 1Password, SOPS-encrypted files); access audited; least privilege.
- [PROD] Rotation process (and ideally automation); short-lived credentials (OIDC federation for CI → cloud, IAM roles, workload identity) over long-lived keys.
- [PROD] Response plan for leaked secrets: revoke, rotate, audit usage.
- Never log secrets; redact in error reports and traces.

## 6. Cryptography & data protection [MVP]
- TLS 1.2+ (prefer 1.3); valid certs with automated renewal.
- Passwords: Argon2id / scrypt / bcrypt with appropriate cost — via a vetted library.
- Encryption at rest (managed disk/DB/storage encryption at minimum); field-level/application-level encryption for highly sensitive fields [PROD/ENT].
- Key management via KMS/HSM; key rotation; envelope encryption [PROD].
- Cryptographically secure randomness for tokens/IDs.
- Constant-time comparison for secrets/signatures.
- Hashing ≠ encryption ≠ encoding.

## 7. File handling [MVP]
- Size limits; type validation by content; randomized storage names; no user-controlled paths (path traversal).
- Store outside the web root; serve with safe headers or from a separate origin.
- Malware scanning for user uploads shared with others [PROD].
- Careful with archive extraction (zip slip, zip bombs), image parsing (decompression bombs), SVG (scriptable), document parsers (XXE).

## 8. Infrastructure & cloud security [PROD]
- Network segmentation: private subnets for data stores; no public DB endpoints; security groups least-privilege.
- IAM least privilege; no root/owner account use; MFA for all humans; separate accounts/projects per environment.
- Infrastructure as code with security scanning (Checkov, tfsec/Trivy, KICS).
- Container security: minimal base images, non-root user, read-only filesystem where possible, no secrets baked into images, image scanning, signed images [PROD/SCALE].
- Kubernetes: RBAC, network policies, pod security standards, secrets encryption [SCALE].
- WAF / DDoS protection for public endpoints [PROD].
- Cloud posture monitoring (CSPM), audit logs (CloudTrail etc.) enabled and retained [ENT].
- Bastion/SSO-based access to production; just-in-time access; session recording for privileged access [ENT].

## 9. Secure SDLC [PROD]
- Code review required for all changes; security-sensitive areas require knowledgeable reviewers (CODEOWNERS).
- Automated: SAST (Semgrep, CodeQL), SCA (Dependabot, Renovate, Snyk, OSV-Scanner), secret scanning (gitleaks, trufflehog), IaC scanning, container scanning, DAST for web apps [SCALE].
- Branch protection; signed commits [ENT]; CI/CD pipeline hardening (pinned actions by SHA, least-privilege tokens, no secrets exposed to forked PRs).
- Security champions / training [SCALE].
- Penetration tests before major launches and periodically [ENT]; bug bounty / vulnerability disclosure program [SCALE/ENT].
- `SECURITY.md` with reporting process ([21](21-documentation.md)).

## 10. Abuse & fraud prevention [MVP]
- Rate limits on login, signup, password reset, OTP, invite, search, export, expensive operations.
- Bot protection / CAPTCHA where abuse is likely.
- Account enumeration resistance (generic messages, consistent timing).
- Email/SMS pumping protection; spend caps on paid APIs (SMS, LLM).
- Content moderation & reporting for user-generated content [PROD].
- Disposable email / signup fraud controls where relevant.

## 11. Security monitoring & response [PROD]
- Log security events: logins (success/failure), MFA changes, permission changes, API key creation, admin actions, data exports ([13](13-observability.md), [18](18-saas-enterprise.md)).
- Alerts on anomalies: brute force, privilege escalation, unusual exports, impossible travel [SCALE].
- Incident response plan including breach notification obligations ([16](16-operations-incidents.md), [10](10-privacy-compliance.md)).

## Red flags
- Secrets in repo history (even if deleted later — rotate them).
- `eval`, dynamic code execution, unsafe deserialization of untrusted data.
- Authorization only in the frontend or only by route, not by object.
- Disabled TLS verification "temporarily".
- Wildcard CORS with credentials; `*` IAM permissions.
- Admin panels exposed publicly without SSO/MFA.
- Verbose errors with stack traces in production.
- Tokens in query strings; PII in logs.
- Hand-rolled crypto/auth.

## Decisions to record
Threat model · auth & session design · secret management approach · encryption approach · security tooling in CI · accepted risks.

## Evidence of done
- Secret, dependency, and static scans run in CI and are green (or have triaged exceptions).
- Authorization tests for object-level and tenant isolation.
- Security headers verified.
- Threat model exists for production systems and is current.
- `SECURITY.md` exists for public repos.

## Related
[09](09-identity-access-tenancy.md) · [10](10-privacy-compliance.md) · [19](19-ai-llm-features.md) · [20](20-dependencies-supply-chain.md) · [checklists/security-baseline.md](../checklists/security-baseline.md)

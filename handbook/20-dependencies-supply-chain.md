# 20 — Dependencies, Supply Chain & Licensing

Every dependency is code you didn't write but are responsible for: its bugs, vulnerabilities, license, maintenance, and transitive dependencies. Add them deliberately, keep them current, and know what you ship.

**Applies when:** adding, upgrading, or removing packages, base images, CI actions, or third-party services; publishing artifacts; open-sourcing.

## Key questions
- Is this dependency necessary, or is it a few lines of code / already in the stdlib or the project?
- Is it healthy, secure, and compatibly licensed?
- How do we learn about vulnerabilities and update promptly?
- Can we prove what's inside our artifacts?

## 1. Adding a dependency [ALL]
Evaluate before adding:
- **Need**: does the stdlib, the framework, or an existing project dependency already cover it? Is it trivial to write (with tests)?
- **Health**: maintenance activity, release cadence, number of maintainers (bus factor), issue responsiveness, adoption, docs quality.
- **Security**: known CVEs, security policy, past incidents, OpenSSF Scorecard.
- **Size & transitive tree**: how many dependencies does it pull in? Bundle impact for frontends.
- **License** compatibility (below).
- **API stability** and upgrade history (frequent breaking majors?).
- **Exit cost**: how hard to replace later? Wrap in an adapter if critical and replaceable.
- Exactly one library per concern in a codebase (one HTTP client, one date library, one validation library).

## 2. Pinning & reproducibility [ALL]
- Commit lockfiles; install with frozen lockfile in CI.
- Pin toolchain versions; pin container base images by digest [PROD]; pin CI actions by commit SHA [PROD].
- Use version ranges in **libraries** (for consumer flexibility) but lockfiles in **applications**.
- Private registry/proxy or mirror for resilience and control [SCALE/ENT].

## 3. Updating [MVP]
- Automated update PRs (Renovate, Dependabot) with grouping and schedules; auto-merge for low-risk patches with passing tests [PROD].
- Security updates prioritized with SLAs by severity (e.g. critical within days) [PROD/ENT].
- Regular major-version upgrade cadence — falling far behind turns upgrades into projects.
- Read changelogs for majors; test thoroughly.
- Remove unused dependencies regularly (depcheck, knip, deptry, `go mod tidy`, cargo-udeps).

## 4. Vulnerability management [MVP]
- SCA scanning in CI (Dependabot alerts, OSV-Scanner, Snyk, Trivy, Grype, `npm audit`, `pip-audit`, `cargo audit`, `govulncheck`).
- Triage: reachable/exploitable? fix, mitigate, or accept with documented reason and expiry.
- Container image scanning; OS package updates via base image refresh.
- Monitor running production artifacts, not only the repo [SCALE].

## 5. Supply chain attacks [PROD]
- Threats: typosquatting, dependency confusion, compromised maintainer accounts, malicious install scripts, compromised CI actions, protestware.
- Mitigations: verify package names; scoped/private package namespaces; disable or review install scripts where possible; lockfile review on changes; minimal CI permissions; isolated build environments; verify signatures/provenance (Sigstore, npm provenance, SLSA).
- Be wary of AI-suggested package names — verify the package exists and is the intended one (hallucinated packages are an attack vector, "slopsquatting").

## 6. SBOM & provenance [ENT]
- Generate SBOMs (CycloneDX or SPDX) for releases (Syft, cdxgen, build tools).
- Build provenance attestations (SLSA levels); signed artifacts (cosign).
- Make SBOMs available to customers on request; VEX statements for non-exploitable CVEs.

## 7. Licensing [ALL for anything distributed]
- Know your project's license and your dependencies' licenses.
- Categories:
  - **Permissive** (MIT, BSD, Apache-2.0, ISC): few obligations — attribution, license text; Apache-2.0 includes patent grant and NOTICE handling.
  - **Weak copyleft** (LGPL, MPL-2.0, EPL): obligations for modifications to the library files/linking terms.
  - **Strong copyleft** (GPL): derivative works distributed must be GPL.
  - **Network copyleft** (AGPL): obligations triggered by network use — significant for SaaS.
  - **Source-available / non-OSI** (BSL, SSPL, Elastic License, Commons Clause, FSL): usage restrictions; check carefully.
  - No license = all rights reserved; don't use.
- Automated license checks in CI with an allow/deny list (license-checker, FOSSA, ScanCode, `cargo deny`, `go-licenses`) [PROD].
- Attribution/NOTICE file generation for distributed software.
- Assets (fonts, icons, images, datasets, model weights) have licenses too.
- AI-generated code: be aware of provenance concerns; avoid reproducing large verbatim copyrighted code.

## 8. Third-party services (SaaS dependencies) [PROD]
- Inventory of external services, owners, data shared, SLAs, costs, and credentials.
- Health monitoring and fallback behaviour ([06](06-reliability.md)).
- Vendor risk review for critical/sensitive vendors ([10](10-privacy-compliance.md)).

## Red flags
- A dependency added for a one-line function.
- Multiple libraries doing the same job.
- No lockfile, or lockfile ignored in CI.
- Hundreds of open Dependabot alerts nobody triages.
- AGPL/SSPL dependencies in a proprietary SaaS without legal review.
- Unpinned `@latest`/`main` references in CI or Dockerfiles.
- Packages suggested by AI installed without verification.

## Decisions to record
Dependency policy · license allow-list · update cadence & SLAs · SBOM/provenance requirements.

## Evidence of done
- Lockfile committed; CI uses frozen installs.
- SCA & license checks pass in CI.
- New dependency justified in PR description.
- SBOM generated for releases (enterprise stage).

## Related
[08](08-security.md) · [14](14-delivery-cicd.md) · [22](22-open-source.md)

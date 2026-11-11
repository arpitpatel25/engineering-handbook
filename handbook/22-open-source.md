# 22 — Open Source & Community

Publishing code is a product decision. A credible open-source project signals quality through its first impression, governance, security posture, and how it treats contributors — not just its code.

**Applies when:** making a repository public, accepting external contributions, publishing packages, or using open source as a portfolio/credibility signal.

## Key questions
- Why open source this (adoption, credibility, community, business model)?
- Which license matches those goals?
- What's the maintenance commitment, and is it stated honestly?
- Is anything sensitive in the code or history?

## 1. Before going public [ALL]
- **Scrub history**: secrets, internal hostnames, customer data, private keys, internal URLs, employee info. If a secret was ever committed, rotate it — rewriting history isn't enough.
- Run secret scanners over the full history (gitleaks, trufflehog).
- Remove proprietary assets you don't have rights to.
- Confirm you own (or your employer permits) releasing the code.
- Check dependency licenses are compatible with your chosen license ([20](20-dependencies-supply-chain.md)).

## 2. License choice [ALL]
- **MIT / BSD / ISC**: maximum adoption, minimal obligations.
- **Apache-2.0**: permissive with explicit patent grant; common for enterprise-friendly projects.
- **MPL-2.0**: file-level copyleft; middle ground.
- **GPL-3.0**: strong copyleft for distributed software.
- **AGPL-3.0**: copyleft extended to network use; deters SaaS competitors, also deters some enterprise adoption.
- **Source-available (BSL, FSL, ELv2)**: protects commercial interests; not OSI open source — be clear about that.
- **Open core**: permissive core + proprietary/enterprise features; define the boundary clearly.
- Docs/content often use Creative Commons licenses.
- Contributor terms: DCO (sign-off) vs CLA — CLAs add friction but enable relicensing.

## 3. First impression (what evaluators look at in 60 seconds) [ALL]
- README per [21](21-documentation.md): clear value prop, screenshot/demo, quickstart that works.
- CI badge green; recent commits; tagged releases; changelog.
- Clean structure; consistent code style; tests present and passing.
- Architecture doc and ADRs (show the thinking).
- Honest project status (experimental / beta / stable / maintained / archived).
- Issue tracker that's triaged, not a graveyard.

## 4. Community health files [MVP]
`LICENSE`, `README.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md` (e.g. Contributor Covenant), `SECURITY.md`, `SUPPORT.md`, `GOVERNANCE.md` [SCALE], `MAINTAINERS`/`CODEOWNERS`, issue templates (bug, feature), PR template, `FUNDING.yml` (optional).

## 5. Contribution workflow [MVP]
- Labels like `good first issue` and `help wanted`; contribution guide with dev setup.
- CI runs on fork PRs **safely** (no secrets exposed to untrusted code; careful with `pull_request_target`).
- Review SLAs you can honestly keep; bots for stale issues used sparingly.
- Discussions forum or chat for questions vs issues for bugs.
- Recognize contributors (all-contributors, release notes credits).

## 6. Security for public projects [MVP]
- Vulnerability disclosure process (`SECURITY.md`, GitHub private vulnerability reporting).
- Branch protection, required reviews, 2FA for maintainers.
- Signed releases and provenance for published packages; trusted publishing (OIDC) to registries.
- Dependabot/Renovate and CodeQL enabled.
- OpenSSF Scorecard and Best Practices badge as quality signals [SCALE].

## 7. Releases & distribution [MVP]
- Semantic versioning; changelog; release notes with upgrade guidance for breaking changes.
- Publish to the ecosystem's registry (npm, PyPI, crates.io, Maven Central, Docker Hub/GHCR, Homebrew) with consistent naming.
- Support/version policy (which versions get fixes).
- Deprecation process with notice periods.

## 8. Governance & sustainability [SCALE]
- Decision-making process (BDFL, maintainers council, foundation).
- Roadmap visibility.
- Funding (sponsors, open collective, commercial offering).
- Bus factor: more than one maintainer with publish rights.

## 9. Showcasing AI-assisted engineering [ALL]
If the project is a showcase of directing AI well, make the process visible:
- `AGENTS.md`/`CLAUDE.md` showing the conventions and constraints given to agents.
- ADRs showing human decisions and rejected alternatives.
- Specs/plans for features (e.g. `docs/specs/`).
- A "How this was built" section: workflow, review process, verification standards, what was human-decided vs agent-implemented.
- Clean, reviewable commit history in small slices.
- Evidence of verification (tests, evals, load tests, security scans) — not just generated code.

## Red flags
- Secrets in git history.
- No license (legally unusable).
- Quickstart that fails.
- Hundreds of unanswered issues; abandoned without an archive notice.
- CI exposing secrets to fork PRs.
- Overstated claims ("production-ready", "blazing fast") without evidence.

## Decisions to record
License · contribution terms (DCO/CLA) · governance · support policy · what's open vs proprietary.

## Evidence of done
- History scanned clean; health files present; quickstart verified from a clean clone; CI green on public runners.

## Related
[20](20-dependencies-supply-chain.md) · [21](21-documentation.md) · [23](23-working-with-ai-agents.md)

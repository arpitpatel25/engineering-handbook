# 14 — Delivery: CI/CD, Environments & Releases

Delivery is how code becomes value safely. The aim: small changes, automatically verified, deployed frequently, observable, and quickly reversible.

**Applies when:** setting up a repo, CI, deployment, environments, versioning, release processes, feature flags.

## Key questions
- How long from merge to production? How long to roll back?
- What is verified automatically before a change reaches users?
- Are environments reproducible and consistent?
- Can we release a feature independently of deploying code?
- What's the blast radius of a bad deploy?

## 1. Source control workflow [ALL]
- **Trunk-based development** with short-lived branches is the default for continuous delivery; GitFlow suits scheduled/versioned releases (e.g. shipped software) but slows feedback.
- Small PRs; one concern per PR; descriptive titles; linked issue/spec.
- **Conventional Commits** (`feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`; `!` for breaking) enabling automated changelogs/versioning [MVP].
- Branch protection: required reviews, required status checks, no force-push to main, linear history or merge policy chosen [MVP].
- CODEOWNERS for sensitive areas [PROD].
- PR template ([templates/pull-request.md](../templates/pull-request.md)).
- Pre-commit hooks (format, lint, secret scan) — fast; CI is the authority.

## 2. Continuous integration [MVP]
Typical pipeline stages (fail fast, cheapest first):
1. Install with lockfile (frozen); cache dependencies.
2. Format check, lint, type check.
3. Unit tests.
4. Build artifacts.
5. Integration tests (containers), contract tests.
6. Security: secret scan, SCA, SAST, IaC/container scans, license check ([08](08-security.md), [20](20-dependencies-supply-chain.md)).
7. E2E tests (possibly against preview env).
8. Publish artifacts (images, packages) with provenance/SBOM [PROD].

Practices:
- Same commands locally and in CI (`make test`, `just ci`, task runner).
- Reproducible builds: pinned toolchain versions (`.tool-versions`, `.nvmrc`, `rust-toolchain`, devcontainer), lockfiles committed.
- CI time budget (~10–15 min); parallelize, cache, split.
- Pin third-party CI actions by commit SHA; least-privilege CI tokens; OIDC to cloud instead of long-lived keys [PROD].
- Monorepos: affected-only builds/tests.

## 3. Build artifacts [MVP]
- **Build once, deploy many**: the same immutable artifact (container image digest, package) promoted through environments; config injected per environment.
- Tag artifacts with commit SHA and version.
- Container images: multi-stage builds, minimal/distroless base, non-root, pinned base digests, scanned, signed (cosign/Sigstore) [PROD].
- SBOM generation (Syft, CycloneDX) and build provenance (SLSA) [ENT].

## 4. Environments [MVP]
- Typical: local → (preview per PR) → staging → production. Fewer is fine early; parity matters more than count.
- **Dev/prod parity**: same OS, runtime, DB engine/version, and backing service types.
- Local dev: one command to start (`docker compose up`, devcontainer, Tilt, Nix), seeded data, documented.
- Staging gets production-like config and data shape (anonymized), not production data.
- Environment configuration in code/IaC; secrets from secret manager ([08](08-security.md)).
- Separate cloud accounts/projects per environment [PROD].

## 5. Continuous delivery / deployment [MVP → PROD]
- Automated deploys from main (continuous deployment) or one-click promotion (continuous delivery).
- Deployment strategies:
  - **Rolling**: replace instances gradually — simple; mixed versions during rollout.
  - **Blue-green**: switch traffic between two full environments — fast rollback; double capacity briefly.
  - **Canary / progressive**: small % of traffic first, monitored, then expand; automated analysis and rollback (Argo Rollouts, Flagger, cloud-native) [SCALE].
  - **Recreate**: downtime; only for non-critical/internal.
- **Rollback** path tested and fast (redeploy previous artifact); DB migrations backward-compatible so rollback is safe ([03](03-data.md)).
- Post-deploy smoke tests and automated health verification.
- Deploy markers in observability ([13](13-observability.md)).
- GitOps (Argo CD, Flux) for declarative, auditable Kubernetes deployments [SCALE].
- Deploy windows/approvals only where risk/regulation requires ([ENT] change management) — prefer automation and small batches over gates.

## 6. Feature flags [PROD]
- Decouple **deploy** (code in production) from **release** (users see it).
- Types: release toggles (short-lived), experiment flags (A/B), ops toggles/kill switches (long-lived), permission/entitlement flags (plan-based) ([18](18-saas-enterprise.md)).
- Targeting by user, tenant, percentage, environment.
- Tooling: LaunchDarkly, Unleash, Flagsmith, GrowthBook, OpenFeature-compatible SDKs, or simple DB-backed flags.
- **Flag hygiene**: owner and expiry for each release flag; remove after rollout; test both paths while live; avoid nested flag logic.

## 7. Versioning & releases [MVP]
- **Semantic Versioning** for libraries, APIs, and shipped software (MAJOR.MINOR.PATCH); CalVer is an option for apps.
- Automated release notes/changelog from commits (release-please, semantic-release, changesets, git-cliff).
- `CHANGELOG.md` following Keep a Changelog for user-facing projects.
- Tagged releases; signed tags [ENT].
- Release checklist for shipped software (mobile/desktop/libraries): see [24](24-project-type-addenda.md).

## 8. Database & data changes in delivery [PROD]
- Migrations run automatically as a deploy step (before new code for expand; after for contract), idempotent, with timeouts and lock awareness ([03](03-data.md)).
- Long backfills run as background jobs, not blocking deploys.

## 9. DORA metrics [SCALE]
Track: deployment frequency, lead time for changes, change failure rate, time to restore service (+ rework rate). Use to improve the system, not to rank people.

## Red flags
- Manual production deploys via SSH/console clicking.
- Different artifacts built per environment.
- "Works on my machine": undocumented local setup.
- CI > 30 minutes; flaky CI ignored.
- Long-lived feature branches; giant PRs.
- No tested rollback; migrations that break the previous version.
- Stale feature flags accumulating.
- CI secrets accessible to fork PRs; unpinned third-party actions.

## Decisions to record
Branching model · CI stages and gates · deploy strategy · environment set · flag tooling · versioning scheme · release process.

## Evidence of done
- CI runs all gates on every PR; main is always deployable.
- One-command local setup documented and working.
- Deploys are automated and rollbacks rehearsed.
- Artifacts are immutable and traceable to commits.

## Related
[03](03-data.md) · [08](08-security.md) · [12](12-testing-quality.md) · [15](15-infrastructure-cost.md) · [20](20-dependencies-supply-chain.md)

# 21 — Documentation & Repository Hygiene

Documentation is the interface between the code and every future reader — teammates, users, auditors, and AI agents. Write the minimum that answers real questions, keep it next to the code, and keep it true.

**Applies when:** creating a repo, making decisions, adding features with user-facing or operational impact, onboarding people or agents.

## Key questions
- Can someone new get the project running in under 15 minutes from the README?
- Can someone find **why** a decision was made?
- Can on-call operate the system from the runbooks?
- Is anything documented that is now false?

## 1. Documentation types (Diátaxis framework) [MVP]
- **Tutorials** — learning-oriented, step by step (getting started).
- **How-to guides** — task-oriented (how to rotate keys, add a tenant).
- **Reference** — information-oriented (API reference, config reference, CLI flags) — generate from code/specs where possible.
- **Explanation** — understanding-oriented (architecture, design rationale, concepts).
Keep them separate; mixing types makes docs hard to use.

## 2. Repository essentials

| File | Purpose | Stage |
|---|---|---|
| `README.md` | What/why, screenshot, quickstart, prerequisites, commands, configuration, links to deeper docs. | ALL |
| `AGENTS.md` / `CLAUDE.md` | Instructions for AI agents: commands, conventions, boundaries, gotchas ([23](23-working-with-ai-agents.md)). | ALL |
| `.env.example` | Every config variable with comments and safe defaults. | ALL |
| `LICENSE` | Legal terms (required for open source). | ALL (public) |
| `docs/architecture.md` | System overview, diagrams (C4 context/container), key flows. | MVP |
| `docs/adr/` | Architecture Decision Records ([templates/adr.md](../templates/adr.md)). | MVP |
| `CONTRIBUTING.md` | Dev setup, workflow, coding standards, PR process, how to run tests. | MVP (shared) |
| `CHANGELOG.md` | User-visible changes per version (Keep a Changelog). | MVP (shipped) |
| `SECURITY.md` | How to report vulnerabilities; supported versions. | MVP (public) |
| `docs/runbooks/` | Operational procedures per alert/incident type ([templates/runbook.md](../templates/runbook.md)). | PROD |
| `CODEOWNERS` | Ownership and review routing. | PROD |
| `CODE_OF_CONDUCT.md` | Community standards. | Open source |
| `SUPPORT.md`, issue/PR templates | Support channels; structured contributions. | Open source |
| API docs | Generated from OpenAPI/proto/GraphQL schema. | MVP (APIs) |
| `docs/project-profile.md` | Stage, risk, constraints ([templates/project-profile.md](../templates/project-profile.md)). | MVP |

## 3. README quality [ALL]
A strong README answers, in order:
1. What is this and who is it for (one paragraph)?
2. Why does it exist / what makes it different?
3. Visual (screenshot, GIF, diagram).
4. Quickstart: prerequisites + ≤ 3 commands to run locally.
5. Usage examples.
6. Configuration reference (or link).
7. Architecture overview (or link).
8. Development: test, lint, build commands.
9. Deployment (or link).
10. Contributing, license, status/badges (CI, coverage, version).

## 4. Architecture Decision Records [MVP]
- One file per significant decision: context, decision, alternatives considered, consequences (positive and negative), status (proposed/accepted/superseded).
- Immutable once accepted; supersede with a new ADR rather than editing history.
- Short (1 page). Written when the decision is made.
- Includes decisions **not** to do something.

## 5. Code-level documentation [ALL]
- Public APIs/modules: doc comments with contract (inputs, outputs, errors, side effects).
- Inline comments explain *why*, constraints, and non-obvious trade-offs; link to issues/ADRs.
- No comments restating the code; no commented-out code.
- Examples in docs are tested (doctests, example tests) where tooling allows [SCALE].

## 6. Docs as code [MVP]
- Docs live in the repo, reviewed in PRs alongside code changes ("docs updated?" in the PR template).
- Diagrams as code (Mermaid, PlantUML, Structurizr, D2) so they're diffable.
- Lint docs (markdownlint, Vale, link checkers) [SCALE].
- Docs site generation (Docusaurus, MkDocs Material, Starlight, mdBook) for larger/public projects.

## 7. Keeping docs true [PROD]
- Generate reference docs from source of truth (schemas, CLI definitions, config schemas).
- Delete stale docs — wrong docs are worse than none.
- Ownership: each doc area has an owner.
- Periodic review of onboarding docs by actually following them (or having an agent follow them in a clean environment).

## 8. Repository hygiene [ALL]
- `.gitignore` appropriate to stack; no build artifacts, secrets, or large binaries committed (Git LFS if needed).
- `.editorconfig`, formatter and linter configs committed.
- Consistent task runner (`Makefile`, `justfile`, `package.json` scripts, `Taskfile`) with standard verbs.
- Clean history: meaningful commits (Conventional Commits), no "wip/fix/fix again" on main (squash or tidy) [MVP].
- Tags/releases for versions.
- Issue labels and templates [MVP shared].
- Archive/deprecate notices for abandoned repos.

## Red flags
- README that only says the framework's default text.
- Setup instructions that don't work on a clean machine.
- Decisions living only in chat threads or someone's head.
- Docs in a wiki disconnected from the code that drifts.
- Massive generated docs nobody reads; AI-generated filler documentation.

## Decisions to record
Docs location/tooling · ADR process · changelog process.

## Evidence of done
- README quickstart works from a clean clone.
- ADR added for significant decisions in the change.
- Changelog/API docs updated for user-visible changes.
- Runbooks exist for new alerts.

## Related
[22](22-open-source.md) · [23](23-working-with-ai-agents.md) · [templates/](../templates/)

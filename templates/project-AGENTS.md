<!--
Copy into your project root as AGENTS.md (and create CLAUDE.md containing "@AGENTS.md").
Keep it SHORT and CURRENT — it is loaded into every agent session. Delete sections that don't apply.
-->
# AGENTS.md — <Project name>

## Project profile
- **What:** <one line>
- **Stage:** <prototype | MVP | production | scale | enterprise>  · **Risk:** <worst realistic defect outcome>
- **Type:** <web service / SaaS / library / CLI / …>
- **Key constraints:** <e.g. must run on X; compliance Y; budget Z>
- Full profile: `docs/project-profile.md`

## Engineering handbook
For design, architecture, and quality decisions, consult the engineering handbook at `<path or URL, e.g. ~/engineering-handbook>` — start with its `AGENTS.md`. This file and the project's ADRs take precedence over it.

## Commands
| Task | Command |
|---|---|
| Setup | |
| Run dev | |
| Test (all) | |
| Test (single file) | |
| Lint / typecheck / format | |
| Migrations (create / apply) | |
| Build | |

Always run lint, typecheck, and relevant tests before claiming a task is done, and report the actual output.

## Architecture map
- `<path>` — <what lives here>
- Module boundaries: <rules, e.g. "features may import from shared/, never from each other's internals">
- Reference implementation to imitate: `<path to a well-structured feature>`
- Decisions: `docs/adr/`

## Conventions
- Errors: <how domain errors are defined and mapped>
- Validation: <library; validate at boundaries>
- Data access: <repository pattern / ORM usage; tenant scoping rule>
- Logging: <logger, required fields, never log PII/secrets>
- Tests: <framework, location, naming, factories>
- Commits: <Conventional Commits>

## Hard rules
- Never commit secrets; config via env (documented in `.env.example`).
- Never bypass the authorization layer; every query is tenant/owner-scoped.
- No new dependencies, infrastructure, or services without asking.
- No edits to generated files: `<paths>`.
- Migrations must be backward compatible with the previous release.
- Don't weaken or delete tests to make them pass.
- Ask before destructive or outward-facing actions (deploys, data deletion, publishing).

## Gotchas
- <non-obvious trap and how to handle it>

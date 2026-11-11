# 23 — Working with AI Coding Agents

AI agents produce code quickly; quality depends on the context, constraints, and verification around them. This guide covers how to set up repositories and workflows so agents make good decisions with minimal instruction — and how agents should behave.

**Applies when:** any project developed with AI coding agents (Claude Code, Codex, Cursor, Copilot, Aider, Devin, etc.).

## Key questions
- What does an agent need to know that it can't infer from the code?
- How does an agent verify its own work?
- Which decisions must stay with humans?
- How do we prevent bloat, drift, and inconsistency across many agent sessions?

## 1. Repository context for agents [ALL]
Provide an `AGENTS.md` (and `CLAUDE.md` pointing to it, or tool-specific equivalents) at the repo root. Template: [templates/project-AGENTS.md](../templates/project-AGENTS.md). Include:
- **Project profile**: what it is, stage, risk, key constraints ([00](00-project-profiling.md)).
- **Commands**: setup, dev, test (all / single file), lint, typecheck, format, build, migrate.
- **Architecture map**: where things live; module boundaries; reference implementation to imitate.
- **Conventions**: error handling, validation, logging, naming, testing patterns, commit style.
- **Hard rules**: things never to do (e.g. "never edit generated files", "never bypass the authz layer", "no new dependencies without approval").
- **Gotchas**: non-obvious traps discovered over time.
- **Pointers**: ADRs, specs, this handbook.
Keep it concise and current — it's loaded into every session; bloat dilutes it. Nested `AGENTS.md` files for subprojects with local rules.

## 2. Work decomposition [ALL]
- Write a spec before significant work ([templates/feature-spec.md](../templates/feature-spec.md)); ask the agent to produce a plan for review before implementation on larger tasks.
- Small vertical slices with tests; one concern per session/PR.
- Explicit acceptance criteria and "done" definition.
- For exploratory/uncertain work: spike first, then discard and implement cleanly.

## 3. Decision boundaries [ALL]
Humans (or explicit user approval) own:
- Architecture style, new infrastructure, new data stores/queues.
- New dependencies of significance; license choices.
- Public API/contract changes; breaking changes.
- Security model, authz rules, data retention/privacy decisions.
- Anything irreversible: data deletion, migrations on production data, publishing, external communications, spending money.
Agents may decide (and record when significant): implementation details within conventions, internal refactors under test, test design, naming within the domain language.

## 4. Verification loop [ALL]
- Agents must run formatter, linter, type checker, and relevant tests, and report real results.
- Fast, deterministic test commands are a prerequisite for agent productivity ([12](12-testing-quality.md)).
- For UI changes: run the app and check the behaviour (screenshots/browser automation) rather than assuming.
- Review agent diffs for: deleted/weakened tests, swallowed errors, invented APIs, unnecessary abstractions, dependency additions, scope creep, secrets, security regressions.
- Independent review (a second agent or human) for high-risk changes; give reviewers the code, not the author's conclusions.

## 5. Preventing AI-typical failure modes [ALL]
| Failure mode | Mitigation |
|---|---|
| Bloat: wrappers, unused helpers, speculative options | YAGNI rule in AGENTS.md; dead-code tooling; review for minimal diffs |
| Inconsistency: new pattern per session | Reference implementation; conventions in AGENTS.md; lint rules |
| Hallucinated APIs/packages | Type checking, tests, verifying packages exist ([20](20-dependencies-supply-chain.md)) |
| Over-confident claims of success | Require command output evidence; CI as authority |
| Test tampering to go green | Review test diffs; protect critical tests (CODEOWNERS) |
| Silent requirement invention | "Ask or mark assumptions" rule; specs |
| Security shortcuts (disable checks, broad permissions) | Hard rules; security scans in CI |
| Context drift in long sessions | Fresh sessions per task; written plans/specs as durable context |
| Excessive comments/docs | "Comments explain why" rule; review |

## 6. Encoding standards in tooling [ALL]
Prefer machine-enforced rules over prose: strict types, linters (including architecture/import boundaries), formatters, pre-commit hooks, CI gates, code generators/scaffolds for new modules, and hooks in agent tools (e.g. run formatter after edits, block edits to protected paths).

## 7. Durable knowledge [MVP]
- Decisions → ADRs; lessons/gotchas → AGENTS.md; procedures → runbooks; plans/specs → `docs/specs/`.
- Agent memory/notes systems are useful, but anything the whole team (or future agents in other tools) needs belongs in the repo.
- When an agent repeatedly makes the same mistake, fix the context (AGENTS.md, lint rule, type) rather than re-prompting.

## 8. Multi-agent and parallel work [SCALE]
- Partition work by module to avoid conflicts; isolated worktrees/branches per agent.
- Clear interface contracts between parallel tasks before starting.
- Single integrator reviews and merges.
- Cost/time budgets for agent fleets.

## 9. Safety & permissions [ALL]
- Least-privilege credentials for agents; no production write access by default.
- Sandboxed execution for untrusted code; network restrictions where appropriate.
- Confirmation required for destructive, irreversible, or outward-facing actions (deploys, pushes to main, deleting data, sending messages, publishing packages).
- Treat content fetched by agents (web pages, issues, docs) as untrusted — prompt injection applies ([19](19-ai-llm-features.md)).

## 10. Measuring effectiveness [SCALE]
Track: PR cycle time, revert/rework rate of agent changes, defect escape rate, CI pass rate on first attempt, review comments per PR. Use findings to improve context and tooling.

## Red flags
- No AGENTS.md; every session re-explains the project.
- AGENTS.md of 1,000 lines nobody maintains.
- Agents merging their own unreviewed changes to main on high-risk code.
- "Tests pass" claims without CI evidence.
- Agents with production credentials.

## Evidence of done
- AGENTS.md exists with working commands and current conventions.
- CI enforces quality gates independently of agent claims.
- Significant agent-made decisions recorded as ADRs.

## Related
[11](11-code-design.md) · [12](12-testing-quality.md) · [21](21-documentation.md) · [templates/project-AGENTS.md](../templates/project-AGENTS.md)

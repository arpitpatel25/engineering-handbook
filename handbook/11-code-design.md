# 11 — Code Design & Repository Structure

Good code is easy to read, easy to change safely, and hard to misuse. For AI-assisted development this matters doubly: agents navigate by names, structure, and locality — a well-structured codebase makes agents dramatically more accurate and less wasteful.

**Applies when:** writing or refactoring code, setting up a repository, establishing conventions, reviewing.

## Key questions
- Can a newcomer (human or agent) find where a behaviour lives within minutes?
- Are boundaries explicit and enforced?
- Is the same thing done the same way everywhere?
- Is every abstraction earning its keep?
- Can business logic be tested without the framework, DB, or network?

## 1. Repository layout [ALL]
- **Organize by feature/domain (vertical slices)**, not by technical layer: `billing/`, `users/`, `projects/` each containing their handlers, services, data access, and tests — over global `controllers/`, `models/`, `services/`, `utils/` folders. Locality reduces context needed for any change.
- Shared kernel kept small and genuinely shared (`shared/`/`platform/`: logging, config, db, http client, errors).
- Clear entry points (`main`, `server`, `worker`, `cli`).
- Monorepo vs polyrepo:
  - **Monorepo**: atomic cross-cutting changes, shared tooling, easier refactors; needs build tooling (Nx, Turborepo, Bazel, Pants, workspaces) at scale.
  - **Polyrepo**: independent lifecycles and permissions; harder cross-repo changes and version drift.
- Conventional top-level: `README.md`, `AGENTS.md`/`CLAUDE.md`, `docs/` (incl. `adr/`), `scripts/`, `infra/` or `deploy/`, `.github/`, config files, `Makefile`/`justfile`/task runner with standard commands (`setup`, `dev`, `test`, `lint`, `build`).

## 2. Layering & boundaries [MVP]
- Separate concerns: **transport** (HTTP/CLI/queue handlers) → **application/use-cases** (orchestration, transactions, authz) → **domain** (business rules, entities, invariants) → **infrastructure** (DB, external APIs, queues) via interfaces/ports.
- Patterns: Hexagonal (Ports & Adapters), Clean Architecture, Onion — same idea: dependencies point inward toward the domain.
- Proportionality: for simple CRUD, thin layers are fine; for rich domains, invest in a real domain layer. Don't build four layers for a single-table admin tool.
- Handlers are thin: parse/validate → call use case → map result/error to response.
- No framework/ORM types leaking into the domain where it hurts testability.
- Enforce boundaries with tooling where possible: module visibility, import linting (dependency-cruiser, eslint-plugin-boundaries, import-linter, ArchUnit, Go internal packages).

## 3. Design principles [ALL]
- **Cohesion high, coupling low.** Things that change together live together.
- **SOLID** as heuristics, not dogma; especially single responsibility and dependency inversion at boundaries.
- **KISS / YAGNI**: no speculative generality, plugin systems, or config options without a current need.
- **DRY with judgment**: duplicate twice before abstracting (rule of three); wrong abstractions cost more than duplication.
- **Composition over inheritance.**
- **Make illegal states unrepresentable**: types, enums, discriminated unions, value objects (Email, Money, TenantId) instead of raw primitives where it adds safety.
- **Parse, don't validate**: convert untrusted input into trusted typed objects at the boundary; inside, trust the types.
- **Immutability by default**; isolate side effects.
- **Pure functions for business logic** where practical — trivial to test.
- **Explicit over implicit**: avoid magic globals, monkey-patching, hidden side effects in getters/constructors.
- **Dependency injection** (constructor/function parameters; container optional) for testability and swappability.
- **Fail fast** on programmer errors and invalid configuration.

## 4. Errors [MVP]
- Distinguish **expected domain errors** (not found, validation, conflict, insufficient funds) from **unexpected failures** (bugs, infra).
- Typed/structured errors (error classes, result types) with stable codes; map to transport (HTTP status, exit code) in one place.
- Never swallow errors silently; either handle meaningfully or propagate with context.
- Add context when wrapping (what was being attempted, relevant IDs) without leaking secrets/PII.
- Don't use exceptions for control flow in hot paths (language-dependent).
- Consistent retry-ability signalling (transient vs permanent).

## 5. Configuration [ALL]
- Config from environment (12-factor), validated and typed at startup; fail fast on missing/invalid values.
- One config module; no `process.env`/`os.getenv` scattered through code.
- Sensible dev defaults; no production defaults that are insecure.
- Secrets separate from non-secret config ([08](08-security.md)).
- Document every variable (`.env.example` with comments).

## 6. Naming & readability [ALL]
- Names from the domain language; precise and unabbreviated; consistent vocabulary (don't mix `fetch`/`get`/`load`/`retrieve` for the same concept).
- Functions do one thing at one level of abstraction; small but not fragmented.
- Avoid boolean parameters that switch behaviour; use separate functions or options objects.
- Comments explain **why** (constraints, trade-offs, links to ADRs/issues), not **what**. Delete commented-out code.
- No vague module names: `utils`, `helpers`, `common`, `misc`, `manager`, `handler2`.
- Consistent file naming convention.
- Public APIs (module exports) documented with their contracts; internal details unexported.

## 7. Types & static analysis [ALL]
- Use the strictest reasonable settings (TypeScript `strict`, mypy/pyright strict, `-Wall -Werror`, Rust clippy, Go vet/staticcheck).
- Formatter (Prettier, Black/Ruff, gofmt, rustfmt) — no style debates.
- Linter with agreed rules; zero-warning policy in CI.
- Avoid `any`/`Any`/unchecked casts; when unavoidable, isolate and document.
- Runtime schema validation at external boundaries (Zod, Valibot, Pydantic, JSON Schema, protobuf) — types alone don't validate data from the network.

## 8. Concurrency & resource handling [MVP]
- Explicit ownership of resources (connections, files, locks) with guaranteed release (`using`, `with`, `defer`, RAII, try/finally).
- Bounded concurrency (semaphores, worker pools) — never unbounded parallel fan-out.
- Avoid shared mutable state; if needed, protect it and document invariants.
- Cancellation/timeouts propagated (context, AbortSignal, cancellation tokens).

## 9. Avoiding bloat (critical with AI-generated code) [ALL]
- Every file, function, abstraction, dependency, and config flag must be justified by current use.
- Delete dead code, unused exports, stale feature flags, and unused dependencies (knip, ts-prune, vulture, deadcode, `go vet`/staticcheck unused).
- No wrapper layers that only forward calls.
- No duplicate utilities reinventing standard library or existing project helpers — search before writing.
- No defensive code against impossible states that the type system already rules out.
- Keep diffs minimal and focused; no drive-by reformatting mixed with logic changes.
- Watch metrics: file size, function length, cyclomatic complexity, duplication — as signals, not targets.

## 10. Refactoring & technical debt [PROD]
- Refactor in separate commits/PRs from behaviour changes, under test coverage.
- Track deliberate debt with context (issue + `TODO(ticket): why`), not anonymous TODOs.
- Strangler-fig pattern for replacing legacy components incrementally.
- Periodic dependency and dead-code cleanup.

## 11. Code review [MVP]
See [checklists/code-review.md](../checklists/code-review.md). Reviews focus on correctness, design, security, tests, and clarity — style is the formatter's job.

## 12. Making code agent-friendly [ALL]
- Colocated features; descriptive file names; small files; consistent patterns agents can imitate.
- An `AGENTS.md`/`CLAUDE.md` at repo root with commands, conventions, boundaries, and pointers ([23](23-working-with-ai-agents.md)).
- Nested `AGENTS.md` in large subdirectories with local conventions.
- Fast, deterministic tests and a single command to run them, so agents can verify their work.
- Types and schemas as machine-readable documentation.
- Example/reference implementations ("golden" module) that new code should mimic.

## Red flags
- God classes/modules; files > ~500–800 lines without reason.
- Business rules in controllers, views, SQL strings, or UI components.
- `utils.ts` with 60 unrelated functions.
- Interfaces with exactly one implementation and no test seam purpose.
- Copy-pasted logic across modules diverging silently.
- Config read from env all over the code.
- Catch-all exception handlers that log and continue.
- Mixed conventions for the same concern (three HTTP clients, two ORMs, two date libraries).

## Decisions to record
Repo layout · layering approach · error handling convention · validation library · formatting/linting setup · module boundary enforcement.

## Evidence of done
- Formatter, linter, type checker run in CI with zero warnings.
- Boundary rules enforced by tooling (where available).
- New code follows the reference module's patterns.
- No dead code/unused deps introduced (tooling check).

## Related
[12](12-testing-quality.md) · [21](21-documentation.md) · [23](23-working-with-ai-agents.md) · [checklists/code-review.md](../checklists/code-review.md)

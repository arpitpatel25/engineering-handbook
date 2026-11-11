# 12 — Testing & Quality

Tests are the safety net that lets you — and AI agents — change code confidently. The goal is not coverage numbers but **fast, trustworthy feedback** on the behaviours that matter.

**Applies when:** writing any code that will be changed again; fixing bugs; refactoring; setting up CI.

## Key questions
- Which behaviours, if broken, would hurt users or the business most?
- How fast is the feedback loop (seconds for unit, minutes for full suite)?
- Do tests catch real regressions, or only break on refactors?
- Are tests deterministic?
- Can an agent run the full relevant test set with one command?

## 1. Strategy [ALL]
- **Test pyramid / trophy**: many fast unit tests for logic; solid integration tests for wiring with real dependencies; few E2E tests for critical journeys. Proportions depend on the app (logic-heavy → more unit; integration-heavy CRUD → more integration).
- Test **behaviour through public interfaces**, not implementation details — so refactors don't break tests.
- Risk-based: deepest testing where defects are costliest (money, auth, data integrity, permissions, migrations).
- Every bug fix gets a regression test that fails before the fix.
- **TDD** (red → green → refactor) is especially effective with AI agents: the test pins the intended behaviour before code is generated.

## 2. Test types

| Type | Purpose | Stage |
|---|---|---|
| **Unit** | Pure logic, edge cases, domain rules. Milliseconds. | ALL |
| **Integration** | Code + real DB/queue/cache (via containers, e.g. Testcontainers); repositories, migrations, transactions. | MVP |
| **API / component** | Service through its HTTP/gRPC interface with external deps faked. | MVP |
| **Contract** | Provider/consumer agreement (Pact, schema validation against OpenAPI/proto). | PROD/SCALE |
| **End-to-end** | Critical user journeys through the real UI/stack (Playwright, Cypress, Detox, Maestro). | MVP (few) |
| **Authorization** | Each role × resource × action; cross-tenant denial. | MVP |
| **Migration** | Up/down on realistic data; compatibility with previous app version. | PROD |
| **Property-based / fuzz** | Invariants across generated inputs (Hypothesis, fast-check, QuickCheck, go-fuzz); parsers, serializers, financial math. | PROD |
| **Snapshot / golden** | Stable outputs (rendered UI, generated files). Use sparingly; review diffs. | MVP |
| **Visual regression** | UI appearance (Chromatic, Percy, Playwright screenshots). | SCALE |
| **Accessibility** | axe-core automated checks + manual audits. | MVP |
| **Performance / load** | See [07](07-performance-scalability.md). | PROD |
| **Security** | SAST/DAST/dependency scans; authz tests; abuse cases. | MVP |
| **Resilience / chaos** | Failure injection. See [06](06-reliability.md). | SCALE |
| **Mutation testing** | Measures test effectiveness (Stryker, mutmut, PIT). | SCALE |
| **Smoke / synthetic** | Post-deploy and continuous production checks. | PROD |
| **LLM evals** | Quality of AI outputs. See [19](19-ai-llm-features.md). | MVP if AI |

## 3. Test quality [ALL]
- **Deterministic**: no reliance on real time (inject clocks), randomness (seed), network, ordering, or shared state. Quarantine and fix flaky tests promptly — flakiness destroys trust.
- **Isolated**: each test sets up its own data; parallel-safe (transactions rolled back, unique schemas/tenants per test).
- **Readable**: Arrange-Act-Assert / Given-When-Then; descriptive names stating behaviour (`rejects_refund_exceeding_original_amount`).
- **Fast**: unit suite in seconds; full suite in minutes; slow tests tagged separately.
- **Meaningful assertions**: assert outcomes, not that mocks were called (unless the call *is* the behaviour, e.g. "email sent").
- **Test data builders/factories** over giant shared fixtures.
- **Mock at boundaries you own** (adapters to third parties), not deep internals; prefer fakes/in-memory implementations for stable semantics.
- Don't test the framework or library; test your use of it.

## 4. Coverage [MVP]
- Coverage is a **signal for untested areas**, not a goal. High coverage with weak assertions is false confidence.
- Track trends and require coverage on changed lines rather than absolute thresholds, if used.
- 100% on critical domain logic is reasonable; elsewhere use judgment.

## 5. Test environments & data [MVP]
- Local test dependencies via containers (docker compose / Testcontainers) — same DB engine and version as production (not SQLite substituting for Postgres).
- Seed data scripts for dev and E2E; no production PII.
- Ephemeral preview environments per PR for E2E/manual QA [SCALE].
- Third-party sandboxes or recorded fixtures (VCR-style) with periodic live verification.

## 6. CI integration [MVP]
- All tests run on every PR; merge blocked on failure.
- Fast checks first (lint, types, unit) → slower (integration, E2E).
- Parallelization and caching to keep CI under ~10–15 min.
- Test results and artifacts (screenshots, traces, logs) retained for debugging failures.
- Nightly/scheduled runs for long suites (soak, full E2E, mutation).

## 7. Quality gates beyond tests [MVP]
- Formatter, linter, type checker, security scans ([14](14-delivery-cicd.md)).
- Code review ([checklists/code-review.md](../checklists/code-review.md)).
- Definition of Done: tests, docs, observability, flags, migration plan, accessibility as relevant.
- Manual exploratory testing for UX-heavy changes.

## 8. Testing with AI agents [ALL]
- Agents must run tests and report actual results; never claim tests pass without running them.
- Ask for tests that would fail without the change (verify by reasoning or by temporarily reverting).
- Beware agents "fixing" failing tests by weakening assertions or deleting tests — review test diffs carefully.
- Beware tests that mirror the implementation (tautological tests).
- Keep a single documented command per test tier so agents can verify quickly.

## Red flags
- Tests that pass with the feature code deleted.
- Mock-heavy tests asserting internal call sequences.
- Flaky tests "retried until green" as policy.
- SQLite in tests, Postgres in prod (behaviour differences).
- E2E suite covering everything, unit tests nothing — slow and brittle.
- No authorization denial tests.
- Disabling/skipping tests to merge.

## Decisions to record
Test strategy & tooling per tier · CI gating rules · coverage policy · test data approach.

## Evidence of done
- New behaviour covered by tests at the appropriate tier, including failure/edge cases.
- Bug fixes include regression tests.
- Full suite passes in CI; no new flaky tests.
- Test commands documented in README/AGENTS.md.

## Related
[11](11-code-design.md) · [14](14-delivery-cicd.md) · [19](19-ai-llm-features.md) · [23](23-working-with-ai-agents.md)

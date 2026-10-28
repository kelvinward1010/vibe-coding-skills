---
name: vibe-test
description: Make the test suite complete for a change — unit, integration and e2e at the right levels, every acceptance criterion and edge case covered, diff coverage measured, flaky tests caught, and a traceability report. Use after implementation, before code review, or when the user asks to "test đầy đủ", "viết test", "tăng coverage", "add e2e".
---

# Vibe Test — prove every requirement, not just the happy path

Input: the diff + `.vibe/PLAN.md` + `.vibe/CONTEXT.md`.
Output: new/updated tests + `.vibe/TEST-REPORT.md`.

`vibe-dev` already wrote a test per task (TDD). This stage is the hardening pass: find what's missing and fill it.

## 0. Test setup

Use the project's existing test stack (from CONTEXT.md). If a level is missing entirely (e.g. no e2e runner), propose the conventional tool for the stack (Vitest/Jest, Pytest, PHPUnit, Go test, Playwright…) and **ask before adding it** — it's a new dependency.

## 1. Build the test matrix

For every acceptance criterion and every *Review focus* item in PLAN.md, decide which level proves it:

| Level | Proves | Use for |
|---|---|---|
| **Unit** | One function/component in isolation | Business logic, validation, pure transforms, reducers, utils |
| **Integration** | Pieces working together with real-ish deps | API endpoint + DB (test DB / container), service + repo, component + store, auth middleware |
| **E2E** | A real user flow in a real browser | The 1–3 critical flows the change touches (Playwright) |

Pyramid: many unit, fewer integration, few e2e. Don't e2e what a unit test can prove.

## 2. Cover each behavior along these dimensions

- **Happy path** — the main expected result.
- **Boundaries** — 0, 1, max, max+1; empty string/list; very long input; unicode/emoji; whitespace.
- **Invalid input** — null/undefined, wrong type, missing fields, malformed format → correct validation error.
- **Error paths** — DB/network failure, timeout, 4xx/5xx from dependencies, partial failure → handled, no data corruption, user sees a useful message.
- **Auth & permissions** — unauthenticated, wrong role, accessing another user's resource → denied.
- **State & concurrency** — double submit, race between two updates, retry/idempotency, stale data.
- **Regression** — every bug fixed gets a test that reproduces the original symptom.
- **UI** (if touched) — loading, empty, error, success states render; form validation; keyboard flow in e2e.

Skip a dimension only if it truly can't apply, and say why in the report.

## 3. Write good tests

- Test **behavior through the public interface**, not internals. Refactoring the code shouldn't break the test.
- Name tests as the behavior: `rejects expired token with 401`, not `test2`.
- Arrange–Act–Assert; one behavior per test; assertions specific (exact value/shape, not just "truthy").
- **Mock only at boundaries** (network, clock, randomness, third-party APIs). Don't mock the code under test or your own modules just to make it easy.
- **Deterministic:** fake timers/fixed clock, seeded randomness, no real network, no `sleep` — wait on conditions. Each test sets up and cleans its own data; order-independent.
- Use factories/fixtures for test data; follow the project's existing test style.

## 4. Measure

1. **Run the full suite** — all green, quote the output.
2. **Diff coverage** — run coverage and check the *changed* lines/branches: target **≥ 90% lines, ≥ 80% branches** on changed code (or the project's own threshold if stricter). Don't chase 100% on trivial code; every uncovered changed branch must be either tested or explained.
3. **Mutation check** — for each core rule, invert a condition / change a boundary / delete a line, run the relevant tests, restore. Tests still green → a test is missing; write it.
4. **Flakiness** — run new/changed tests 3× (or with `--repeat`/shuffle if supported). Any inconsistency → fix the cause, never retry-to-green.

## Rules

- Never delete, skip (`.skip`, `xit`, `@pytest.mark.skip`), or weaken an assertion to get green. A failing test means a bug or a wrong test — find out which.
- If a test reveals a real bug, fix it in the code (back to `vibe-dev`), not in the test.
- Don't snapshot huge outputs as a substitute for assertions.

| Rationalization | Reality |
|---|---|
| "Coverage is 85%, good enough" | Coverage shows what ran, not what was checked. Mutation check tells you. |
| "Happy path works, edge cases are rare" | Users hit edge cases daily; that's where bugs live. |
| "Mocking everything makes it fast" | Then you're testing the mocks. Mock only at boundaries. |
| "It's flaky, just re-run" | Flaky = nondeterminism bug in the test or the code. Fix it. |
| "E2E is slow, skip it" | One e2e on the critical flow catches wiring bugs units never will. |

## Output: `.vibe/TEST-REPORT.md`

```markdown
# Test Report — <feature> (<date>)

Suite: `<command>` → <n> passed, 0 failed (quoted)
Diff coverage: lines <x>% · branches <y>%
Mutation check: <n> mutants, <k> caught → <added tests>
Flakiness: new tests run 3× → stable

## Traceability
| Requirement | Unit | Integration | E2E | Status |
|---|---|---|---|---|
| AC1 user can filter by status | `filter.test.ts` | `GET /candidates?status` | `candidates.spec.ts` | ✅ |
| Focus: 10k rows | — | `pagination.int.test.ts` | — | ✅ |

## Edge cases covered
## Not covered (and why)
## Bugs found while testing
```

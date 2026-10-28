---
name: vibe-dev
description: Implement an approved plan task by task — test-first, matching project conventions, with fresh verification evidence before ticking anything done. Use when the user says "implement", "code it", "dev đi", or after a plan in .vibe/PLAN.md is approved.
---

# Vibe Dev — build it, prove it

Input: `.vibe/PLAN.md` (approved) + `.vibe/CONTEXT.md`.

## Loop, per task

1. **Read** every file the task touches before editing it.
2. **Red** — write the test from the plan. Run it. It must fail, for the reason the plan says. A test that passes before the code exists proves nothing.
3. **Green** — make the smallest change that passes. Match surrounding code: naming, comment density, error handling, imports.
4. **Refactor** — clean up only what this task introduced. Re-run tests.
5. **Verify** — run the task's verify command (see the gate below).
6. **Tick** the task in `PLAN.md` (`- [x]`). If the plan was wrong, update it and add a line under *Decisions* — never silently diverge.

For a long plan with independent tasks, you may hand each task to a fresh subagent (task text + CONTEXT.md only), then review its diff yourself before ticking. A subagent's "done" is not evidence — the diff and test output are.

## Verification gate (no exceptions)

Before claiming anything works, is fixed, passes, or is done:

1. **Identify** the command that proves it.
2. **Run** it fresh and complete — not an earlier run, not a subset.
3. **Read** the full output: exit code, failure count.
4. **Claim only what the output shows**, and quote the evidence.

| Claim | Required evidence |
|---|---|
| Tests pass | Test output with 0 failures, from this turn |
| Build OK | Build command exit 0 (lint passing ≠ build passing) |
| Bug fixed | Test reproducing the original symptom now passes |
| Requirement met | Each acceptance criterion checked line by line |
| Subagent finished | `git diff` shows the change + tests ran |

**Red flags — stop and run the check:** "should work", "probably", "seems to", feeling done, about to commit, trusting a report you didn't verify.

## Rules

- No scope creep. Unrelated issues go into `PLAN.md` → *Out of scope → found during dev*.
- No new dependency without a line in *Decisions*. One dependency per change.
- Never commit secrets. Never skip, delete, or weaken tests/lint rules to get green.
- Keep each change reviewable: aim ~100 lines, split if it heads past ~300.
- Don't commit or push unless the user asks.

| Rationalization | Reality |
|---|---|
| "I'll add tests at the end" | Tests written after the code test what you wrote, not what was asked. |
| "Too small to need a test" | Small changes break things too; the test is small as well. |
| "Linter passed, so it's fine" | A linter isn't a compiler or a test suite. |
| "Just this once" | That's what every skipped check said. |

## Finish

Run the full build + test suite once. Report the actual output. If something fails and you can't fix it, stop and explain — don't hand off a red build as done.

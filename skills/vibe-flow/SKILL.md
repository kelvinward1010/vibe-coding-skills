---
name: vibe-flow
description: Run the full vibe coding pipeline for a feature or fix — read the codebase, write a plan, implement it test-first, complete the test suite, review the code, then audit the UI/UX. Use when the user asks to build a feature end to end, says "vibe code this", or wants the whole read → plan → dev → test → review → audit cycle.
---

# Vibe Flow — orchestrator

Runs the six stages in order. Each stage writes one file into `.vibe/` at the project root, and the next stage reads it. This keeps context small and lets the user stop, edit, or resume at any stage.

| # | Stage | Skill | Output |
|---|-------|-------|--------|
| 1 | Read | `vibe-read` | `.vibe/CONTEXT.md` |
| 2 | Plan | `vibe-plan` | `.vibe/PLAN.md` |
| 3 | Dev | `vibe-dev` | code + TDD tests + ticked checklist in `PLAN.md` |
| 4 | Test | `vibe-test` | full test suite + `.vibe/TEST-REPORT.md` |
| 5 | Review | `vibe-review` | `.vibe/REVIEW.md` |
| 6 | UI/UX audit | `vibe-uiux-audit` | `.vibe/UIUX-AUDIT.md` (skip if no UI change) |

## Pick the track first

| Track | When | Stages |
|---|---|---|
| **Quick** | Typo, copy change, 1-file fix with an obvious answer | Dev (with a test + verification gate) → short Review. No plan file. |
| **Standard** | Normal feature or bug | All six stages. |
| **Spike** | User wants to try something fast to learn | Build throwaway code, no tests/plan/review. Label it **spike**. Keeping it = new request through the Standard track. |

State the track you picked in one line. If a Quick task grows, stop and switch to Standard. Any code that ships has tests — only a spike may skip them.

## Rules

1. **Gate after Plan.** Show the plan summary and wait for approval before stage 3. Everything else runs without stopping unless something is ambiguous or risky.
2. **Resume, don't restart.** If `.vibe/` files exist, read them and continue from the first unfinished stage. Ask before overwriting an existing `PLAN.md`.
3. **Loop on findings.** A bug found in Test, Critical/Required in Review, or High in Audit → back to Dev, fix, re-run Test, then re-run that stage once. Report anything still open.
4. **Fresh eyes for review.** Run stages 5–6 in a fresh subagent when the same session wrote the code.
5. **Skip what doesn't apply.** No UI touched → skip stage 6 and the e2e part of stage 4, and say so.
6. Add `.vibe/` to `.gitignore` unless the user wants these files committed.

## Final report

- What was built (1–3 lines)
- Files changed
- Tests: suite result + diff coverage — quoted from a fresh run, not assumed
- Open issues from Test/Review/Audit, if any
- Next step: commit / open PR / keep iterating (ask; don't do it unprompted)

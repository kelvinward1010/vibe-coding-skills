---
name: vibe-plan
description: Turn a feature request or bug into a concrete, ordered implementation plan with acceptance criteria and test-first tasks. Use after reading the codebase, when the user asks to "plan", "lên plan", "break this down", or before any non-trivial change.
---

# Vibe Plan — decide before you code

Input: the user's request + `.vibe/CONTEXT.md` (run `vibe-read` first if it doesn't exist).
Output: `.vibe/PLAN.md`, written for an implementer who has never seen the codebase or the request.

**Hard gate:** nothing gets built (code, scaffolding, installing deps) until the user approves the plan. Reading and exploring are fine.

## 1. Clarify (only what's missing)

- **Look before asking.** Never ask what CONTEXT.md or the code already answers.
- If the request is complete, skip to step 2.
- Otherwise restate the request in your own words (≤ 150 words) and ask **one question at a time** — the one whose answer would most change the build. Explain why it matters if that's not obvious.
- If the user says "just guess", propose something concrete and label it as your call.

## 2. Pick an approach

Size the work first:
- **Small** (1–3 files, no new concepts) → one approach, short plan.
- **Feature** → offer 2–3 approaches with trade-offs, recommend one with reasons, let the user choose.
- **Several independent subsystems** → split into separate plans, each shipping working, testable software. Do them in order.

Reuse existing patterns from CONTEXT.md. A new pattern needs a written reason.

## 3. Write the plan

Map the files first (create / modify / test, one-line responsibility each), then the tasks.

A **task** = the smallest unit with its own test cycle and an independently verifiable result (≈ one commit). Fold setup and docs into the task that needs them.

Each step must be specific enough that the implementer writes exactly one reasonable thing:
- Test steps state the assertion.
- Code steps state exact names, signatures, and values from the request.
- Verify steps state the command **and the expected output**.
- Banned: "TBD", "handle edge cases", "add appropriate error handling", "etc."

## 4. Self-review before showing

- Every acceptance criterion maps to at least one task + test, at the right level in *Test strategy*.
- Names/types are consistent across tasks.
- Every *Review focus* item has a test in its owning task.
- The plan is shorter than the code it describes. If not, it's a transcript, not a plan — cut.

## Output: `.vibe/PLAN.md`

```markdown
# Plan — <feature>

## Goal
## Acceptance criteria
- [ ] AC1 ...
## Constraints        <!-- copied verbatim from the request / CONTEXT -->
## Decisions
- <choice> — because <reason> (user's call / my call)
## Files
| File | Action | Responsibility |
## Review focus       <!-- real-world inputs most likely to break this -->
- empty list / 10k rows / concurrent submit / expired session ...
## Test strategy
| Requirement | Unit | Integration | E2E |
|---|---|---|---|
| AC1 | <what> | <what> | <critical flow?> |

## Tasks
### T1 — <what>
Files: `a.ts` (modify), `a.test.ts` (create)
Interfaces: consumes `X`, produces `fooBar(id: string): Promise<Foo>`
- [ ] Write failing test: <assertion>
- [ ] Run `<cmd>` → expect FAIL: <reason>
- [ ] Implement: <exact change>
- [ ] Run `<cmd>` → expect PASS
- [ ] Commit-sized checkpoint

### T2 — ...

## Risks / rollback
## Out of scope
```

Then show the user a ≤ 5-line summary + the list of decisions marked "my call", and wait for approval.

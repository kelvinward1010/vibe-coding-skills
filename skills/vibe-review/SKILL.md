---
name: vibe-review
description: Review code changes across five axes (correctness, readability, architecture, security, performance) plus test quality and plan conformance — produce ranked, verified findings. Use after implementation, on a diff/branch/PR, or when the user asks for "code review" / "review giúp".
---

# Vibe Review — would a staff engineer approve this?

Input: the diff (`git diff`, branch vs base, or PR) + `.vibe/PLAN.md` if present.
Output: `.vibe/REVIEW.md`.

**Approval bar:** approve when the change clearly improves code health and follows project conventions — even if you'd have written it differently.

**Fresh eyes:** if you wrote the code in this session, run the review in a fresh subagent given only the diff, PLAN.md, and CONTEXT.md. Authors miss their own assumptions.

## Process

1. **Context** — what is the change meant to do? Which acceptance criteria? Anything built that wasn't asked?
2. **Size check** — ~100 changed lines good, ~300 OK if one logical change, ~1000 → recommend splitting (stack, by file group, or vertical slices). Refactors mixed with features → flag.
3. **Tests first** — read `.vibe/TEST-REPORT.md` if present, then check it holds up: do tests exist at the levels the plan named, assert behavior (not implementation), cover edge cases and *Review focus* items? Is diff coverage met?
   **Mutation check:** invert one condition the change adds, run the suite, restore the file. Still green → name the missing test. Missing tests are **Required**, not Optional.
4. **Five axes**, per changed file:
   - **Correctness** — spec match, edge cases (empty/null/huge/concurrent), error paths, off-by-one, async/await, race conditions, broken callers of changed functions.
   - **Readability** — clear names, straight control flow, no needless cleverness. Abstractions earn their complexity. A new conditional bolted onto an unrelated flow = design smell.
   - **Architecture** — follows existing patterns, clean module boundaries, no duplication of existing helpers, no circular deps. A refactor must reduce complexity, not move it. Files heading past ~1000 lines → decompose.
   - **Security** — input validated at boundaries, parameterized queries, output encoded (XSS), authz on new endpoints, no secrets in code/logs, new deps vetted.
   - **Performance** — N+1 queries, unbounded loops/fetches, missing pagination, blocking calls that should be async, unnecessary re-renders, big objects on hot paths.
5. **Verify the verification** — which tests ran, did the build pass, are there screenshots for UI changes? Run them yourself if unsure.

## Rules

- Verify each finding by reading the actual code path. No speculative findings; if unsure, say so.
- Each finding: `file:line`, what's wrong, a concrete failure scenario, a fix.
- Order by leverage: correctness & security → structure → everything else. Don't pad with nits.

| Rationalization | Reality |
|---|---|
| "Tests pass, so it's good" | Tests don't catch architecture, security, or readability problems. |
| "AI-generated code is probably fine" | It's confident and plausible even when wrong — scrutinize more, not less. |
| "We'll clean it up later" | Later rarely comes. Ask for it now. |
| "It's just a version bump" | It's a behavior change nobody here wrote. Read the changelog. |

## Severity

- **Critical** — security flaw, data loss, broken functionality. Blocks ship.
- **Required** — must fix before merge.
- **Optional** — worth considering.
- **Nit** — style; only if cheap.
- **FYI** — no action.

## Output: `.vibe/REVIEW.md`

```markdown
# Review — <feature> (<date>)

Verdict: ✅ ship / ⚠️ ship after Required fixes / ❌ needs rework
Size: <n> lines across <m> files · Tests: <command> → <result>

| # | Sev | File:line | Issue | Fix |
|---|-----|-----------|-------|-----|
| 1 | Critical | `api/user.ts:42` | ... | ... |

## Details
### 1. <title>
Failure scenario: ...
```

## When receiving review feedback (applying fixes)

Check each finding against the code before changing anything. If a finding is wrong, say why with evidence instead of complying. Fix Critical/Required, re-run tests, then re-review only the touched areas.

---
name: vibe-read
description: Read and map a codebase before changing it — stack, structure, conventions, how to run/test, and the code relevant to the task. Use when starting work in an unfamiliar repo, before planning a feature, or when the user asks "understand this project" / "đọc code".
---

# Vibe Read — understand before touching

Goal: produce `.vibe/CONTEXT.md`, a compact map that lets the planner work without re-reading the repo.

## Steps

1. **Identify the stack.** Read manifests (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `composer.json`, …), lockfiles, framework configs, `README`, `CLAUDE.md`, `AGENTS.md`, `.env.example`.
2. **Find the commands.** How to install, run dev server, build, lint, typecheck, test. Verify by reading scripts — don't guess.
3. **Map the structure.** Top-level folders and what each owns. Entry points, routing, data layer, state management, UI component library, styling system.
4. **Extract conventions.** Naming, file layout, error handling, API call pattern, test style, commit style. Quote 1 short example file path for each.
5. **Zoom into the task.** If the user gave a task, locate every file likely touched: grep for related names, follow imports one or two levels. List them with one line on why each matters.
6. **Note risks.** Shared code with many callers, missing tests, generated files, env/secret requirements, anything surprising.

7. **Check the baseline.** Run the test/build commands once and record the result. Failures that exist before you touch anything must not be blamed on the new change later.

For large repos, fan out searches in parallel (Explore agent) and keep only conclusions.

## Output: `.vibe/CONTEXT.md`

```markdown
# Context — <project> (<date>)

## Stack
## Commands
| Task | Command |
## Baseline
<test/build result before any change>
## Structure
## Conventions
## Task-relevant files
- `path/to/file.ts` — why it matters
## Risks & unknowns
```

Keep it under ~150 lines. Facts only, each with a file path where possible.

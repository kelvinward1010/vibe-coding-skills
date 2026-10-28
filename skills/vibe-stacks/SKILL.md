---
name: vibe-stacks
description: Stack packs — stack-specific dev patterns, review checks and test tooling for React/Next.js, Vue, NestJS, FastAPI, Go, PostgreSQL/MySQL, MongoDB, Unity 6 (URP), Godot 4. Loaded by the other vibe skills when the project's `## Vibe` config or detected stack matches. Use when the user asks for stack-specific rules or best practices for one of these stacks.
---

# Vibe Stacks — rules per stack

Reference only; this skill does no work on its own. The other vibe stages load the packs that match the project:

- `vibe-plan` / `vibe-dev` — the **Dev** section
- `vibe-test` — the **Testing** section
- `vibe-review` — the **Review checks** section; a failed check is Required unless the pack marks it Critical

Load only the packs the project uses (from *Stack packs* in the `## Vibe` section of CLAUDE.md, or detected by `vibe-read`). Never load all of them.

| Pack | Applies when |
|---|---|
| [react-nextjs.md](react-nextjs.md) | `react` in package.json; `next` for the Next.js part |
| [vue.md](vue.md) | `vue` or `nuxt` in package.json |
| [nestjs.md](nestjs.md) | `@nestjs/core` in package.json |
| [fastapi.md](fastapi.md) | `fastapi` in pyproject.toml / requirements |
| [go.md](go.md) | `go.mod` |
| [sql.md](sql.md) | PostgreSQL or MySQL — ORM (Prisma, TypeORM, SQLAlchemy, GORM, sqlc…) or raw driver |
| [mongodb.md](mongodb.md) | `mongodb`, `mongoose`, `motor`, `beanie`, or the Go mongo driver |
| [unity.md](unity.md) | `ProjectSettings/ProjectVersion.txt` (Unity 6, URP) |
| [godot.md](godot.md) | `project.godot` with `config_version=5` (Godot 4) |

A project's own *Team rules* override a pack when they conflict.

## Adding a pack

One file per stack, same four sections: **Applies when**, **Dev**, **Review checks**, **Testing**. Keep it under ~60 lines, only rules that are specific to the stack — generic rules already live in the stage skills. Add a row to the table above.

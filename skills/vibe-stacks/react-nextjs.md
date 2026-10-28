# React / Next.js

**Applies when:** `react` in package.json. Next.js sections apply when `next` is present; check `app/` (App Router) vs `pages/` (Pages Router) and follow whichever the project uses.

## Dev

- Function components + hooks only. Derive values during render instead of copying props into state.
- `useEffect` is for syncing with something outside React (subscriptions, DOM, timers). Not for data derivation, not for reacting to user events — do that in the event handler.
- Server data goes through the project's data layer (TanStack Query, SWR, RSC fetch, server actions). Don't hand-roll fetch + `useState` + `useEffect` if a data library is already there.
- Stable, unique `key` from data ids — never array index for lists that reorder, insert, or delete.
- Forms: follow the project's form library (React Hook Form, Formik…) and its schema validator (Zod, Yup). Validate again on the server.

**Next.js (App Router)**
- Components are Server Components by default. Add `"use client"` only to the smallest leaf that needs state, effects, or browser APIs.
- Server-only code (DB, secrets, SDKs with keys) stays in server components, route handlers, or server actions; mark shared modules with `import "server-only"` when the project does.
- Only `NEXT_PUBLIC_*` env vars reach the browser — everything else must never be referenced from client code.
- Pick caching on purpose: `fetch` cache options / `revalidate` / `revalidatePath` / `revalidateTag`. After a mutation, revalidate what it changed.
- Use `next/image`, `next/link`, `next/font`; add `loading.tsx` / `error.tsx` / `not-found.tsx` for new route segments.

## Review checks

- **Critical:** secret or server-only module imported into a `"use client"` file; server action or route handler without auth/authz check; user HTML rendered via `dangerouslySetInnerHTML` without sanitizing.
- Hook rules: hooks called conditionally or in loops; `useEffect` with missing or lying deps (lint disabled to silence it).
- `useEffect` that fetches without cleanup/abort → race condition when inputs change fast.
- State that duplicates props or other state; effects that only set state from other state.
- New context provider high in the tree with a value object created every render → whole subtree re-renders.
- `"use client"` placed on a page/layout when only one child needs it.
- Mutation without revalidation → stale UI after save.
- Large client bundle additions (charting, date, icon libs imported whole) — check for a lighter import or `next/dynamic`.

## Testing

- Unit/component: the project's runner (Vitest or Jest) + React Testing Library. Query by role/label/text, assert what the user sees; don't assert on state or internal props.
- Mock the network at the boundary with MSW rather than mocking hooks or fetch per test.
- Server actions / route handlers: test as plain functions with a real (test) DB or the integration setup from CONTEXT.md.
- E2E: Playwright against `next build && next start` (not `next dev`) for the critical flows; use `getByRole` locators and web-first assertions, no `waitForTimeout`.

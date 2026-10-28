---
name: vibe-uiux-audit
description: Audit UI/UX of changed or specified screens — UI states, usability, visual consistency, accessibility (WCAG 2.2 AA), responsiveness, forms, motion, copy — in the live app and in code. Use after UI work, when the user asks for "UI/UX audit", "check giao diện", "review my UI", or an accessibility/design review.
---

# Vibe UI/UX Audit — would a real user be happy?

Input: changed UI files (from the diff or `PLAN.md`), or screens/files the user names. If neither, ask which.
Output: `.vibe/UIUX-AUDIT.md`.

## Two passes

**1. Live pass (preferred)** — start the dev server (command from CONTEXT.md) and drive it with a browser tool (Playwright / Chrome DevTools MCP):
- Screenshots at **375px, 768px, 1280px**; light + dark if supported.
- Walk the main task end to end with **keyboard only**.
- Check the console for errors/warnings and the network tab for failed or duplicate requests.
- Trigger loading, empty, error, and long-content states (throttle network, empty data, 200-char strings).

**2. Code pass** — read the UI files and apply every rule in [references/web-interface-rules.md](references/web-interface-rules.md).

If the app can't run, do the code pass only and say so in the report.

## What to judge

1. **States** — loading, empty, error, success, disabled, long text/overflow, many items.
2. **Usability** — one clear primary action per screen; labels say what happens; destructive actions confirm or undo; inline validation with the fix in the message; no dead ends; URL reflects filters/tabs/pagination.
3. **Visual consistency** — existing design tokens/components only (no hard-coded colors, ad-hoc spacing, new font sizes); alignment and spacing rhythm; hover/active/focus states stronger than rest.
4. **Accessibility (WCAG 2.2 AA)** — contrast ≥ 4.5:1 text / 3:1 large text & controls; full keyboard operability with visible `:focus-visible` ring; semantic elements before ARIA; labelled inputs; alt text; `aria-label` on icon buttons; targets ≥ 24×24px (44×44 on mobile); no info by color alone; `prefers-reduced-motion` respected.
5. **Responsive** — no horizontal scroll at 375px; tables and long content adapt; modals/drawers fit and contain scroll.
6. **Copy & i18n** — consistent terms and tone; specific button labels; no hard-coded strings if the project uses i18n; dates/numbers via `Intl`.
7. **Perceived performance** — skeletons for slow loads, optimistic updates where sensible, image dimensions set (no layout shift), long lists virtualized.

## Severity

**High** = blocks a task or fails WCAG AA · **Medium** = noticeable friction · **Low** = polish.

## Output: `.vibe/UIUX-AUDIT.md`

```markdown
# UI/UX Audit — <scope> (<date>)

Method: live at <url> + code / code-only
Score: <n>/10 — one-line verdict

## Top issues
| # | Sev | Screen / file:line | Category | Issue | Fix |
|---|-----|--------------------|----------|-------|-----|

## Code findings
### src/Button.tsx
src/Button.tsx:42 - icon button missing aria-label
src/Button.tsx:67 - transition: all → list properties
### src/Card.tsx
✓ pass

## Quick wins
## Evidence (screenshots, console errors)
```

Code findings stay terse: `file:line - issue`, no explanation unless the fix is non-obvious.

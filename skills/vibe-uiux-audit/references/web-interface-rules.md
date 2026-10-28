# Web interface rules (code pass)

Condensed from [Vercel Web Interface Guidelines](https://github.com/vercel-labs/web-interface-guidelines). Apply each rule to the files under audit; report violations as `file:line - issue`.

## Accessibility
- Icon-only buttons have `aria-label`; decorative icons `aria-hidden="true"`
- Form controls have `<label>` or `aria-label`
- `<button>` for actions, `<a>`/`<Link>` for navigation — never click handlers on `<div>`/`<span>`
- Images have `alt` (empty if decorative)
- Async updates announced with `aria-live="polite"`
- Semantic HTML before ARIA; hierarchical headings; skip link on pages with nav

## Focus
- Visible focus on every interactive element; use `:focus-visible`
- No `outline: none` / `outline-none` without a replacement
- `:focus-within` for compound controls; sticky UI never hides the focused element

## Forms
- Correct `type`, `inputmode`, `autocomplete`, meaningful `name`
- Never block paste
- Labels clickable; checkbox/radio + label share one hit target
- Submit stays enabled until the request starts, then shows progress
- Errors inline, next to the field; focus the first error on submit
- Spellcheck off for emails, codes, usernames
- Warn before leaving with unsaved changes

## Animation
- Honor `prefers-reduced-motion`
- Animate only `transform` / `opacity`; no `transition: all`
- Animations interruptible; motion > 5s has pause/stop

## Content handling
- Containers handle short, average and very long input (truncate / clamp / wrap)
- Flex children that truncate have `min-w-0`
- Every list/table has an empty state
- Number columns use `tabular-nums`; loading text ends with `…`

## Images & performance
- `<img>` has explicit `width`/`height`
- Below-fold images `loading="lazy"`; hero images prioritized
- Lists > 50 items virtualized
- No layout reads during render; inputs cheap per keystroke
- Critical fonts preloaded with `font-display: swap`

## Navigation & state
- URL reflects state (filters, tabs, pagination, open panels) → deep-linkable
- Links support Cmd/Ctrl-click and middle-click
- Destructive actions need confirmation or undo

## Touch & layout
- `touch-action: manipulation`; `overscroll-behavior: contain` in modals/drawers
- Gestures have a tap/click and keyboard alternative
- Full-bleed layouts respect `env(safe-area-inset-*)`
- Layout with flex/grid, not JS measurement; no unwanted scrollbars
- Never disable zoom in the viewport meta

## Theming
- Dark theme sets `color-scheme: dark`; `theme-color` matches background
- Native `<select>` has explicit background and color

## Locale
- Dates via `Intl.DateTimeFormat`, numbers/currency via `Intl.NumberFormat` — no hard-coded formats
- Language from `Accept-Language` / `navigator.languages`, not IP

## Hydration (SSR frameworks)
- Inputs with `value` have `onChange` (or use `defaultValue`)
- Date/time rendering guarded against server/client mismatch

## Copy
- Active voice, second person; specific button labels ("Save changes", not "OK")
- Error messages say how to fix it
- Numerals for counts; consistent capitalization per project style

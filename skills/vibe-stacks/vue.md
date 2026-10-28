# Vue / Nuxt

**Applies when:** `vue` in package.json (Vue 3). Nuxt sections apply when `nuxt` is present. Follow the project's style: Composition API with `<script setup>` is the default; don't mix in Options API unless the file already uses it.

## Dev

- `<script setup lang="ts">` with `defineProps` / `defineEmits` typed. Props are read-only — emit an event or use `defineModel` instead of mutating them.
- `computed` for derived values; `watch` only for side effects. Prefer `watch` with an explicit source over `watchEffect` when the dependencies matter.
- `ref` for primitives and values you replace; `reactive` only for objects you never reassign. Destructuring a `reactive` loses reactivity — use `toRefs`.
- Shared logic goes in composables (`useXxx`) that return refs; shared app state in Pinia stores (setup-store style if the project uses it).
- `v-for` always with a stable `:key` from data ids. Never `v-if` and `v-for` on the same element.

**Nuxt**
- Data fetching with `useFetch` / `useAsyncData` (SSR-safe, deduped) — not raw `fetch` in `onMounted` for page data.
- Server code under `server/api/`; secrets only via `runtimeConfig` (private keys), never in `runtimeConfig.public`.
- Browser-only code guarded with `import.meta.client`, `<ClientOnly>`, or `onMounted`.

## Review checks

- **Critical:** `v-html` with user content without sanitizing; secret in `runtimeConfig.public` or client code; `server/api` handler without auth check.
- Prop mutated in a child; store state mutated directly from components when the project uses actions.
- Reactivity lost: destructured `reactive`/props without `toRefs`, ref passed as `.value` into a composable that needs to track it.
- `watch` that sets state another `computed` could derive; deep watchers on large objects.
- Event listeners, intervals, or subscriptions added in `onMounted` without cleanup in `onUnmounted`.
- Hydration mismatch risk in Nuxt: `Date.now()`, `Math.random()`, `window` used during SSR render.

## Testing

- Unit/component: Vitest + Vue Test Utils (`mount`); for Nuxt, `@nuxt/test-utils`. Find elements by role/text, trigger events, assert rendered output and emitted events — not internal refs.
- Pinia: `createTestingPinia` for components; test stores directly as plain units.
- Mock the network with MSW.
- E2E: Playwright against a production build (`nuxt build && nuxt preview` / `vite preview`).

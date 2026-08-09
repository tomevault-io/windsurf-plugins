---
trigger: always_on
description: The admin dashboard is a Svelte 5 SPA in `web/dashboard/`. Every page MUST
---

# GoModel dashboard — Svelte conventions

The admin dashboard is a Svelte 5 SPA in `web/dashboard/`. Every page MUST
follow these rules so the pages compose into one coherent app.

## Golden rules

1. **Preserve the admin API contract.** Keep every admin API endpoint,
   request/response field, query parameter, and behavioral rule exactly as
   the Go backend implements it — `internal/admin/*.go` is the source of
   truth. Do not invent endpoints or payload fields.
2. **CSS lives with its owner.** Component-specific styles are scoped
   `<style>` blocks in the owning component; the shared design system
   (tokens, reset, typography, buttons, forms, tables, alerts, layout,
   keyframes) plus rules that target child-component DOM — a class passed
   as a prop into `Icon` or `LoadingState` renders in the child's markup,
   where the parent's scope hash cannot match — lives in `src/styles/`.
   `dashboard.css` is only the entry point: it `@import`s the modules in
   cascade order, so append new rules to the module that owns them and
   never reorder the imports. No CSS preprocessor, on purpose: scoped
   styles, custom properties, and native nesting cover the need with zero
   extra dependencies. Reuse existing shared class names for standard
   elements.
3. **Mind the scope hash when moving CSS.** Scoping adds specificity, and it
   cuts both ways:
   - Moving a rule *out* of dashboard.css can make it win ties it used to
     lose. Move its state/media/theme variants with it and check the shipped
     computed style is unchanged.
   - A global modifier (`.x-request`) silently *loses* to the scoped base
     (`.x.svelte-hash`) it is meant to override. If the modifier is composed
     dynamically, keep it in the owning component as a compound `:global`
     (`.x:global(.x-request)`) so it still outranks the base.
   - The compiler prunes selectors it cannot see in the markup, so a
     component whose state classes are computed strings (`WorkflowChart`,
     `ProviderStatusCard`) cannot hand that CSS to a child component.
4. **Svelte 5 runes only.** `$state`, `$derived`, `$effect`, `$props`,
   snippets/`{@render}`. No legacy `$:` reactivity, no svelte/store.
5. **Files:** page code lives in `src/pages/<page>/`. The entry component is
   `<Pascal>Page.svelte`. Split big pages into sub-components in the same
   directory (atomic design: compose from
   `$lib/components/atoms|molecules|organisms`). Keep files under ~400 lines
   where practical.
6. **Shared foundation code lives in `src/lib/`** (plus `src/App.svelte`) —
   changes there affect every page, so keep them deliberate. Page-specific
   helpers belong in the page directory.
7. **Do NOT add new npm dependencies.**

## Foundation — use it, don't re-implement

Imports use the `$lib` alias. **Each module documents its own exports and
props in its header comment** — read those for the details; this list exists
so you know what already exists.

### HTTP: `$lib/api/client.js`

```js
const result = await getJSON("/admin/foo?x=1", { label: "foo", signal });
// result = { ok, stale, status, data, res }
const saved = await sendJSON("/admin/foo", "POST", payload, { label: "save foo" });
```

- `result.stale === true` → the response belongs to an old API key: **return
  without touching your state**, or it clobbers fresh data.
- 401s are handled globally (the auth dialog opens); `result.ok` is `false`.
- `errorMessage(result, fallback)` reads a result envelope;
  `errorPayloadMessage(data, fallback)` reads a raw `{error:{message}}` body.
  Both really live in `$lib/api/errors.js`, which imports no Svelte runtime —
  pure page logic and its `node:test` suite import them from there directly.
- `apiFetch(path, options)` is the raw escape hatch (SSE, blobs); it adds auth
  + timezone headers and the base path. Never call `fetch` on `/admin/...`.
- **CRUD stores use `$lib/api/adminCrud.js`**: `loadAdminList` /
  `sendAdminMutation` reduce a request to an outcome object with the guard
  ladder applied in the one correct order (stale first, then
  unavailable-503, then errors; 401 loads stay silent). Apply the outcome
  to your `$state` fields instead of re-implementing the branches.

### Stores (`$lib/stores/*.svelte.js`) — all singletons

`auth` · `router` · `themeStore` (bump `tick` to rebuild charts) · `sidebar` ·
`modals` (owned by the `Modal` atom — don't touch) · `timezone` ·
`runtimeConfig` (feature-flag visibility) · `modelsStore` · `dateRange`
(shared reporting window) · `usageData` · `flash` · `confirmDialog` (typed
confirmations).

### Components

- **atoms** — `Icon` (kebab-case lucide names), `Spinner`, `Modal`,
  `EmptyState`, `NoDataIllustration`, `CopyButton`, `TableActionButton`,
  `DialogCloseButton`, `SegmentedControl`, `EnabledToggle`, `GoModelLogo`.
- **molecules** — `LoadingState`, `Pagination`, `DatePicker`, `FilterInput`,
  `InlineHelpSection`, `ChartCanvas`, `DemoModeBanner`, `FormField`.
- **organisms** — `AuthBanner`, `AuthDialog`, `Sidebar` (nav items in
  `navigation.js`), `ThemeToggle`, `FlashMessages`,
  `TypedConfirmationDialog`, `EditorDialog`.

`Modal` handles Escape/backdrop/scroll-lock and autofocuses
`[data-modal-autofocus]`. `ChartCanvas` runs its `build()` inside an effect,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ENTERPILOT/GoModel](https://github.com/ENTERPILOT/GoModel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

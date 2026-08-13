---
trigger: always_on
description: - Solve problem at hand. No refactors unless asked.
---

# Kittycord Rules

## Philosophy

- Solve problem at hand. No refactors unless asked.
- Smallest diff wins. Three similar lines beat premature helper.
- Less code with same behavior always wins. Equivalent solutions: shorter one ships. Compression is intelligence.
- Don't over-guard. Check only conditions that can actually occur. `if (user)` on non-optional store return is wrong. `try/catch` on code that can't throw is wrong. `?.` on type-guaranteed values is wrong. Dead defensive code isn't safety.
- No comments unless asked. Keep existing ones, they carry context.
- Delete dead code. Don't comment out.
- Natural human text in errors, descriptions, toasts. No dashes, no robotic phrasing. Descriptions start capitalized, end with period.
- One-file plugin default. Split only past ~250 lines or when second file is genuinely reusable (`utils.ts`, `settings.ts`, `components/`).
- No abstraction for 1-2 call sites. Inline beats single-use hook. `createX` used once is just a function call.
- Don't reimplement what `@webpack/common` or `@utils/*` exports. Search first.
- Don't cache Discord data. Stores are source of truth, read on demand. Cache only what isn't in a store.
- Don't manage state Discord manages. `useStateFromStores` over `useState` + flux that mirrors a store.
- Don't recreate Discord components. Reuse `Button`, `TextInput`, `Tooltip`, `Menu.*`, etc. Hand-rolled `<button>` loses theme, focus rings, a11y.
- No state machine for boolean. No class wrapping single function. No `Strategy`/`Manager`/`Handler` suffix unless multiple implementations exist.

## TypeScript

- `strict: true`, `noImplicitAny: false`, `noUncheckedIndexedAccess: false`. Annotate params explicitly. Array index access not auto `undefined`.
- Never `any`. Use `unknown` + narrow, or type webpack results via `find*` generic.
- Never `as unknown as`. Write type guard or use `satisfies`.
- Never `@ts-ignore`. Use `@ts-expect-error` with one-line reason only when unavoidable.
- Never `!` non-null assertion. Narrow with `if` or early return.
- Never `enum` (including `const enum`). Use `as const` object plus literal union.
- Never boxed primitives (`Number`, `String`, `Boolean`, `Object`, `Function`, `{}`).
- Never `namespace`. Modules only.
- Never `../../` when `@alias` exists.
- `import type` for type-only imports, including from `"react"`. Plain `import { ReactNode }` is violation.
- `satisfies T` over `as T` when value already conforms. `as` only for genuine narrowing.

## Plugin Policy

Does NOT get merged:

- Selfbots, automation of user actions, client state spoofing, API abuse.
- Plugins requiring user-supplied API keys or untrusted third-party endpoints.
- Plugins targeting specific third-party bots. Official Discord apps are fine.
- CSS-only plugins that just hide/restyle UI.
- Functionality trivial via built-in plugins or Discord itself.
- New dependencies without strong justification.

## Plugin Structure

- Every NEW source file starts with SPDX header:
  ```
  /*
   * Vencord, a Discord client mod
   * Copyright (c) <year> Vendicated and contributors
   * SPDX-License-Identifier: GPL-3.0-or-later
   */
  ```
  The header text stays exactly `Vencord, a Discord client mod` with author `Vendicated and contributors` on every new file. This is the GPL-3.0 copyright notice required by the license and enforced by the `simple-header/header` lint rule — it is NOT a brand label, so don't change it to "Kittycord".
  Older files carry the full GPL preamble. Leave existing headers alone, don't rewrite them.
- Folder per plugin. New original plugins go in `src/kittycordplugins/<name>/index.tsx`. Other plugin folders exist: `src/plugins/`, `src/equicordplugins/`, `src/moggcordplugins/`. Single-file plugins still get a folder.
- Folder suffixes the build filters on: `.desktop`, `.web`, `.dev`, `.discordDesktop`, `.vesktop`, `.equibop`.
- `export default definePlugin({ ... })`. Never named.
- Required fields: `name`, `description`, `authors`. Prefer the shared `Devs`/`EquicordDevs` maps; inline `{ name, id }` author objects are allowed for ported plugins whose authors aren't in those maps.
- List `dependencies` for every API plugin used.
- Lock environment via folder suffix. Per-setting platform gating via `target?: "WEB" | "DESKTOP" | "BOTH"` on the setting def (not the plugin). Code-path gating via build constants.
- `requiresRestart` defaults true when `patches` exist. Set only to override.
- `startAt?: StartAt` overrides default `WebpackReady`. Values: `Init`, `DOMContentLoaded`, `WebpackReady`.
- `settingsAboutComponent?: React.ComponentType` renders above settings UI.
- `reporterTestable?: ReporterTestable` flags parts the auto-reporter exercises (`Start`, `Patches`, `FluxEvents`, `None`).
- `isModified?: true` marks an inherited upstream plugin that Kittycord has modified.
- Optional: `tags?: PluginTag[]` (closed list in `src/utils/types.ts`), `required` (forced on, hidden toggle), `enabledByDefault` (on at first run, toggleable), `hidden` (off the UI), `searchTerms?: string[]`.

### Declarative over imperative

Use these plugin fields instead of calling `add*`/`remove*` manually. `PluginManager` handles lifecycle and auto-enables the matching API plugin.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KittyCord-Production/Kittycord](https://github.com/KittyCord-Production/Kittycord) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

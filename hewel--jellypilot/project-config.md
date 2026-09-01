---
trigger: always_on
description: Tauri v2 Jellyfin MPV Shim rewrite; external MPV controlled via JSON IPC (no libmpv embed).
---

# AGENTS.md

Tauri v2 Jellyfin MPV Shim rewrite; external MPV controlled via JSON IPC (no libmpv embed).

## Role Separation

**scoped-implementer**

* Only implements the requested changes.
* Does not review or approve its own code.

**reviewer**

* Only finds bugs and explains why the code may fail.
* Does not edit code, write fixes, or implement changes.


## Stack

- **Frontend**: Solid.js + TypeScript + Rsbuild + TanStack Router + TanStack Form + Ark UI + Panda CSS
- **Backend**: Rust (Tauri v2) with tauri-specta for type-safe bindings
- **Data / Effects**: Effect-TS
- **Tools**: Bun, Oxc (Oxlint/Oxfmt), Rstest (jsdom)

## Where to Look

| Task | Location | Notes |
|------|----------|-------|
| Add Tauri command | `src-tauri/src/command.rs` | `#[tauri::command]` + `#[specta]` |
| Register command | `src-tauri/src/lib.rs` | Add to `collect_commands![]` |
| Frontend page | `src/routes/**` | Page-level `.tsx` route code; never split into `-*.tsx` modules |
| Reusable component | `src/components/**` | Non-page `.tsx` components; never in `src/features/**` |
| Tauri data workflow | `src/effects/**` | Reusable loaders/actions via Effect wrappers |
| Design tokens | `panda.config.ts` + `src/styles/theme-tokens.ts` | Panda owns all styling values |
| UI primitives | `src/components/ui/**` | Shared design-system components |
| Rust↔TS bindings | `src/bindings.ts` | Auto-generated (debug builds only); use typed `commands.*`, never raw `invoke()` |
| Add test | `tests/*.test.ts` | Rstest + @testing-library |
| Rust backend | `src-tauri/` | See `src-tauri/AGENTS.md` |

## Commands

```bash
bun run dev          # Start Rsbuild dev server (port 3000)
bun run build        # Production build → dist/
bun run test         # Rstest
bun run test:watch   # Rstest watch mode
bun run check        # Oxfmt/Oxlint + type/Rust checks (includes format + typecheck)
bun run test:e2e     # Reuse an existing native E2E build
bun run e2e:verify  # Full native path plus production-isolation proof

# Tauri (run from project root)
bun tauri dev       # Dev mode with hot reload
bun tauri build     # Production desktop build
```

## Conventions

- **Solid.js**: Use the `solidjs` skill for all Solid-specific patterns.
- **Forms**: Always use `@tanstack/solid-form` with `createForm` for form handling.
- **Styling**: Panda CSS is the only styling system ([ADR 0011](docs/adr/0011-panda-css-styling.md)). Author owner-local `Component.styles.ts` / route `*.styles.ts` modules via `@styled-system/*`; use semantic Panda tokens and colocated `css` / `cva` / `sva`. Supported Tauri sizes: 1280×720 min, 1600×900 default. Stress review widths (review config only): 800×600, 640×720, 360×720. Design system: `docs/design-system.md`.
- **Solid classes**: `class` for static strings; `classList` only for additive conditional classes (properties the base never sets, e.g. `spin`); never to override base styles — concatenated atomic classes resolve conflicts by generated stylesheet order, not call order. For state-driven overrides: prefer a Panda condition in the one style definition when the element publishes the state (`_pressed`, `_open`, `&[data-state=…]`, `[data-state=open] &`); otherwise a `cva` variant; for multiple independent flags contesting one property, merge raws in precedence order (`css(base, flagA && a, flagB && b)` — last wins). `cx` from `@styled-system/css` is the only permitted class joiner and takes no conditionals (no `&&`/ternaries); never author class-name merge helpers.
- **Path aliases**: In `src/**`, import through the tsconfig aliases (`@components/*`, `@bindings`, `@styled-system/*`, `~effects/*`, `~styles/*`, `~utils/*`) instead of cross-directory relative paths. Same-directory imports (e.g. `./Component.styles`) stay relative. `tests/**` use relative imports — Rstest only aliases `@styled-system`.
- **TypeScript / Effect**: All Effect rules live in [docs/agents/effect.md](docs/agents/effect.md). Read and follow it; do not duplicate them here.
- **Route data loading**: Await only critical data; defer slow data as promises behind `<Suspense />` with stable skeletons. Follow [TanStack Router deferred data loading](https://tanstack.com/router/latest/docs/guide/deferred-data-loading).
- **Ark UI Dialogs**: Use standard Ark UI Dialog primitives; no custom ARIA overlays, `onInteractOutside` handlers, or `id` attributes. Controlled dialogs require `<Portal>`; use `lazyMount` + `unmountOnExit`.
- **Style Tests**: Style test policy in [docs/agents/style-tests.md](docs/agents/style-tests.md); read and follow it.
- **Validation**: Proportionate verification policy in [docs/agents/validation.md](docs/agents/validation.md); scale test scope to the change's blast radius and never skip verification.
- **Native E2E**: Agent policy and command contract in [docs/agents/e2e.md](docs/agents/e2e.md). Use focused native E2E whenever acceptance crosses Tauri startup, IPC, post-mount routing, desktop interaction, or sandboxed persistence.

## Git & Command Safety


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hewel/jellypilot](https://github.com/hewel/jellypilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->

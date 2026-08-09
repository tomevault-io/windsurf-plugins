---
trigger: always_on
description: This file is the source of truth for **how to change this repo**. Prefer matching existing patterns over inventing new stacks.
---

# AGENTS.md — Duckling contributor / AI coding guide

This file is the source of truth for **how to change this repo**. Prefer matching existing patterns over inventing new stacks.

## Project snapshot

Duckling is a **Tauri 2** desktop app for browsing files/databases and running SQL.

| Layer | Stack |
|-------|--------|
| Frontend | React 19, TypeScript, Vite 8, path alias `@/` → `src/` |
| UI | Tailwind CSS **v4**, shadcn-style components (**Base UI** / `components.json` style `base-vega`), Lucide icons |
| State | Zustand (persist) + Jotai; dual-write durable config via `@tauri-apps/plugin-store` |
| i18n | **Lingui** (`en` source, `zh-CN`) |
| Editor / grid | Monaco, VisActor VTable |
| Backend | Rust workspace: `src-tauri` (Tauri commands) + `connector` (DB dialects) |
| Package manager | **pnpm only** (`pnpm-lock.yaml`) |

## Commands (always use these)

```bash
pnpm install              # never npm/yarn for app deps
pnpm dev                  # Vite only (web preview)
pnpm tauri dev            # full desktop app
pnpm build                # frontend production build
pnpm tauri build          # installable app
pnpm test                 # Vitest (tests/ + co-located *.test.ts)
pnpm lint                 # tsc -- typecheck
pnpm i18n:extract         # lingui extract --clean
pnpm i18n:compile         # compile catalogs (generated .mjs is gitignored)
```

Rust:

```bash
cargo check --manifest-path src-tauri/Cargo.toml
# or from workspace root after install:
cargo test -p connector   # when changing connector
```

After non-trivial TS changes: run **`pnpm lint`** and relevant **`pnpm test`**.  
After user-visible string changes: **`pnpm i18n:extract`**, fill **zh-CN**, then **`pnpm i18n:compile`**.

## Package management

- Use **pnpm** for all frontend dependencies and scripts.
- Do not introduce `package-lock.json` / `yarn.lock` for the app.
- Prefer existing deps (`radash`, `es-toolkit`, `nanoid`, `zod`, `sonner`) before adding new ones.
- UI primitives: **do not edit** `@/components/ui/*` (upstream shadcn/Base UI). App-specific styling and behavior go in `@/components/custom/ui/*` (and `src/styles.css` for global slot overrides). Do not reintroduce Radix if Base UI already covers the case.

## TypeScript / React conventions

- Path alias: `@/...` (see `vite.config.ts`).
- Prefer functional components and existing store hooks (`useTabsStore`, `useSettingStore`, workspace atoms, etc.).
- **Do not add comments** unless the user asks or the logic is non-obvious and has no better name.
- Match local style: imports, naming, file placement.
- Toasts: `sonner` (`toast.success` / `toast.error`).
- Hotkeys: register in `src/hotkeys/registry.ts` and wire via `useAppHotkey` / Monaco actions as existing code does.

### Layout / flex (common pitfall)

Nested flex + scroll/canvas areas need **`min-h-0 min-w-0 overflow-hidden`** on the chain so children can shrink.  
Result grids (VTable) and split panes are sensitive to missing containment and **subpixel** width; see `tableSizeAntiJitter` on ListTable options when touching tables.

## Tailwind CSS v4

This project uses **Tailwind v4** via `@tailwindcss/vite` and CSS-first config in `src/app/globals.css`.

**Do:**

- Use `@import 'tailwindcss'`, `@theme { ... }`, `@plugin`, `@custom-variant` as in `globals.css`.
- Prefer utility classes already used in the codebase (`size-*`, `min-h-0`, `bg-background`, `text-muted-foreground`, `ring-foreground/10`, etc.).
- Use design tokens / CSS variables from `@theme` and existing shadcn tokens.
- Compose classes with `cn()` from `@/lib/utils`.

**Do not:**

- Add or revive a classic **`tailwind.config.js` / v3-style** config as the source of truth.
- Use deprecated v3-only patterns when a v4 equivalent exists (e.g. avoid inventing new `@tailwind base/components/utilities` pipelines).
- Hard-code one-off colors that ignore theme tokens unless matching nearby code.

## Internationalization (Lingui) — mandatory for UI text

User-visible English strings must go through Lingui.

| Context | API |
|---------|-----|
| JSX text | `<Trans>...</Trans>` from `@lingui/react/macro` |
| Attributes / non-JSX | `const { t } = useLingui()` then `t\`...\`` |
| Module-level labels | `msg\`...\`` from `@lingui/core/macro` + `_(msg)` at render |

Rules:

1. Import macros from **`@lingui/react/macro`** / **`@lingui/core/macro`** (not bare `@lingui/react` for `Trans`/`t`).
2. After adding/changing copy: `pnpm i18n:extract`, translate empty `msgstr` in `src/locales/zh-CN/messages.po`, then `pnpm i18n:compile`.
3. **Commit `.po` files only.** Compiled `src/locales/**/*.mjs` are gitignored.
4. Source locale is **en**; always fill **zh-CN** for new messages (do not leave `msgstr ""` for shipped UI).
5. Prefer clear message text; avoid complex expressions inside macros (extract placeholders cleanly).

## Testing

- Framework: **Vitest** (`pnpm test`). Config lives in `vite.config.ts` (`globals: true`, `environment: 'node'`).
- Place unit tests under **`tests/`** (preferred for stores/libs) or co-located `*.test.ts` when matching existing patterns (`src/ast`, etc.).
- Mock `localStorage` / Tauri `invoke` like existing tests (`tests/tabsStore.test.ts`, `tests/scratchSql.test.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [l1xnan/duckling](https://github.com/l1xnan/duckling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

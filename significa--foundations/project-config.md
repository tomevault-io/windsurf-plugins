---
trigger: always_on
description: Guidelines for agentic coding agents working in this repository.
---

# CLAUDE.md

Guidelines for agentic coding agents working in this repository.

---

## Commands

All commands use **pnpm** (v10.32.1, Node ≥ 22.12.0).

| Command             | Purpose                                                            |
| ------------------- | ------------------------------------------------------------------ |
| `pnpm dev`          | Start Astro dev server (Pagefind/search does not work)             |
| `pnpm dev:pagefind` | Build + index Pagefind + dev server (search works)                 |
| `pnpm build`        | Production build; `postbuild` runs Pagefind indexing automatically |
| `pnpm preview`      | Serve the production build locally                                 |
| `pnpm check`        | **CI gate** — must pass before merging (see below)                 |
| `pnpm lint`         | `biome lint --write --unsafe .`                                    |
| `pnpm format`       | Biome format (non-Astro) + Prettier format (Astro)                 |
| `pnpm types:check`  | `astro sync && tsc -noEmit`                                        |

### CI gate

`pnpm check` runs four things in sequence:

1. `biome ci` — lints and checks formatting for all non-Astro files
2. `prettier --check "**/*.astro"` — checks formatting for Astro files
3. `pnpm types:check` — full TypeScript type check
4. `pnpm check:deps` — verifies every component's `page.mdx` declares the cross-Foundations imports it uses (see "Page dependency discipline" below)

**Always run `pnpm check` after making changes.** Fix all errors before considering a task done.

### No test framework

There is no Jest, Vitest, or Playwright setup. There is no test command.

---

## TypeScript

- Config extends `astro/tsconfigs/strict` — the strictest Astro preset.
- Path aliases: `@/*` → `src/*`, `~/*` → repo root.
- Use `import type` for all type-only imports.
- Avoid `any`. If unavoidable, add a `// biome-ignore lint/suspicious/noExplicitAny: <reason>` comment.
- Prefer `interface` for component props and object shapes; use `type` for unions, mapped types, and simple aliases.
- Component props must extend the underlying HTML element type:
  - `React.ComponentPropsWithRef<'button'>` when forwarding refs
  - `React.ComponentPropsWithoutRef<'div'>` otherwise
  - `React.ComponentProps<typeof SomeComponent>` to inherit another component's props
- Use the React 19 ref-as-prop pattern — do **not** use `forwardRef`.
- Use the React 19 `use()` hook instead of `useContext()` to consume context.
- Pass context directly as JSX: `<MyContext value={ctx}>` (no `.Provider`).
- Use the `satisfies` operator when helpful for type narrowing without widening.
- CSS custom properties on React elements are covered by the global augmentation in `src/react.d.ts` — pass `--token` props inline without type errors.

---

## Code Style

### Formatting rules

|                 | `.ts` / `.tsx` files | `.astro` files |
| --------------- | -------------------- | -------------- |
| Tool            | Biome                | Prettier       |
| Quotes          | Single               | Double         |
| Semicolons      | Always               | Always         |
| Line width      | 80                   | 128            |
| Indent          | 2 spaces             | 2 spaces       |
| Trailing commas | ES5                  | ES5            |
| JSX quotes      | Double               | Double         |

### Imports

- Biome's `organizeImports` runs automatically — do not manually sort imports.
- Use `import type { Foo }` for type-only imports; never mix types and values in a single `import`.
- Always import Phosphor icons from the SSR-safe subpath: `import { IconName } from '@phosphor-icons/react/dist/ssr'`. The root barrel uses a dynamic registry that breaks with Astro SSR and is unreliable under Vite HMR (intermittent `Cannot read properties of undefined (reading 'call')` errors). The `/dist/ssr` build is static ESM, fully tree-shakable, SSR-safe, and HMR-stable.
- Import `cn` and `cva` from `@/lib/utils/classnames`, not directly from their packages.

### Exports

- Use **named exports only**. No default exports, except in `*.preview.tsx` files.
- Export components at the bottom of the file: `export { ComponentName }`.
- For compound components, use `Object.assign` then re-export under the clean name:
  ```ts
  const CompoundDialog = Object.assign(Dialog, { Content: DialogContent, Trigger: DialogTrigger });
  export { CompoundDialog as Dialog };
  ```
- Export types with `export type { MyType }` when the file also has value exports.

---

## Naming Conventions

| Thing                  | Convention                | Example                               |
| ---------------------- | ------------------------- | ------------------------------------- |
| Files & folders        | `kebab-case`              | `use-scroll-lock.ts`, `color-picker/` |
| React components       | `PascalCase`              | `ColorPicker`, `DialogContent`        |
| Hooks                  | `camelCase`, `use` prefix | `useScrollLock`, `useMatchMedia`      |
| Utilities / functions  | `camelCase`               | `composeRefs`, `clamp`                |
| CVA style objects      | `<name>Style`             | `buttonStyle`, `badgeStyle`           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [significa/foundations](https://github.com/significa/foundations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->

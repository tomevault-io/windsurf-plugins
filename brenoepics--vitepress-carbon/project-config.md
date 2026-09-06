---
trigger: always_on
description: Operating guide for AI coding agents (and new contributors) working in this
---

# AGENTS.md

Operating guide for AI coding agents (and new contributors) working in this
repository. Read this before making changes. For anything visual, also read
[design.md](design.md) — the design‑system spec that all UI must follow.

## What this is

**vitepress-carbon** ("Carbon") is a sleek, near‑monochrome theme for
[VitePress](https://vitepress.dev) documentation sites. It is a **pnpm workspace
monorepo**:

| Path                 | Package                      | Purpose                                                                                     |
| -------------------- | ---------------------------- | ------------------------------------------------------------------------------------------- |
| `packages/theme`     | `vitepress-carbon`           | The published theme. **The main package.** Builds to `dist/`.                               |
| `packages/cli`       | `vpcar`                      | Published CLI (`vpcar init`) that scaffolds Carbon sites. Built with `unbuild`.             |
| `packages/demo`      | `vitepress-carbon-demo`      | Demo docs site that consumes the theme via `link:../theme`. Not published.                  |
| `packages/storybook` | `vitepress-carbon-storybook` | Storybook workbench for the theme's components. Not published; deployed to `/storybook/`.   |
| `packages/templates` | —                            | JSON starter descriptors used by the CLI's init flow.                                       |
| `shared/`            | —                            | Code shared by tooling, not shipped: the VitePress harness used by the tests and Storybook. |
| `/` (root)           | `vitepress-carbon-monorepo`  | Private root: shared scripts, tooling config, tests.                                        |

## Environment

- **Package manager: pnpm `9.9.0` only.** Every package has `preinstall: only-allow pnpm` — npm/yarn will fail. Do not add a `package-lock.json` or `yarn.lock`.
- **Node: `22.18.0`** (`.node-version`). Engines allow `^20.19.0 || >=22.18.0`.
- Install with `pnpm install` (use `--frozen-lockfile` in CI‑like runs).

## Commands

Run these from the repo root.

| Task                | Command                                    | Notes                                                     |
| ------------------- | ------------------------------------------ | --------------------------------------------------------- |
| Install             | `pnpm install`                             |                                                           |
| Dev                 | `pnpm dev`                                 | Builds the theme, then serves the demo.                   |
| Storybook           | `pnpm storybook`                           | Component workbench on `:6006`.                           |
| Build all           | `pnpm build`                               | Order: **theme → cli → demo → storybook**.                |
| Build theme only    | `pnpm --filter vitepress-carbon run build` | `vue-tsc --noEmit` → `tsc` → copy assets.                 |
| Test                | `pnpm test`                                | `vp test run` (Vitest).                                   |
| Component tests     | `pnpm test:components`                     | `vp test run __tests__/components`.                       |
| Integration tests   | `pnpm test:integration`                    | `vp test run __tests__/integration`.                      |
| Lint (autofix)      | `pnpm lint`                                |                                                           |
| Lint (check only)   | `pnpm lint:check`                          |                                                           |
| Format (write)      | `pnpm format`                              |                                                           |
| Format (check only) | `pnpm format:check`                        |                                                           |
| Full check          | `pnpm check`                               | `vp check` — the same thing CI runs. Before every commit. |

**Before opening a PR, `pnpm check`, `pnpm test`, and `pnpm build` must all pass.**

These scripts delegate to `vp` with no path arguments so they cover the whole
repo, exactly like CI's `vp check --fix`. Don't narrow them to a glob — a glob
that skips a directory (`__tests__/`, say) lets type errors through locally and
fails in CI instead.

## Toolchain notes (important)

- Linting, formatting, and testing are handled by the **`vp` CLI** (Vite+ /
  `vite-plus`, pinned in the `pnpm-workspace.yaml` catalog to `@voidzero-dev/*`
  `0.1.24`). It bundles **oxlint + tsgolint** and a formatter.
- **There are no `.eslintrc`, `.prettierrc`, or `oxlint.json` files** — rules come
  from `vp` defaults. Don't add competing linters/formatters.
- The `vp` toolchain updates itself and can get stricter over time (e.g. it rejects
  removed TypeScript compiler options). If `pnpm check` fails on a `tsconfig-error`,
  fix the flagged `tsconfig.json`, don't pin around it.
- Git hooks are Vite+‑managed (`.vite-hooks/`); `pre-commit` runs `vp staged`.
  There is no husky or commitlint.

## Conventions

- **Formatting:** 2‑space indent, **single quotes**, no semicolons in `.mjs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brenoepics/vitepress-carbon](https://github.com/brenoepics/vitepress-carbon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->

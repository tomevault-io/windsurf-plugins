---
trigger: always_on
description: Instructions for AI coding agents (Claude Code, Copilot, Cursor, Codex…). Task-specific skills live in `.agents/skills/`; the maintenance directive for this file lives in `.agents/context/`.
---

# Talend/UI — Agent Instructions

Instructions for AI coding agents (Claude Code, Copilot, Cursor, Codex…). Task-specific skills live in `.agents/skills/`; the maintenance directive for this file lives in `.agents/context/`.

## Repository Overview

This is **Talend/UI**, a pnpm workspaces monorepo containing shared front-end libraries for Talend products.

- **Workspaces**: `packages/*`, `tools/*`, `fork/*` (`pnpm-workspace.yaml`)
- **Stack**: React 18, TypeScript 5, Babel 7
- **Build tooling**: shared `@talend/scripts-*` packages (see `tools/`), orchestrated by [Turborepo](https://turbo.build) (`turbo.json`)
- **Tests**: Vitest · **Lint**: oxlint + Stylelint · **Format**: Prettier
- **Versioning**: [Changesets](https://github.com/changesets/changesets) (`@changesets/cli`)
- **Package manager**: pnpm (version pinned in `packageManager` / `.tool-versions`)

Run `pnpm install` at the root, then `pnpm build` (`build:lib` + `build:lib:esm` through turbo).

Common root commands: `pnpm vitest:run`, `pnpm oxlint:run`, `pnpm stylelint:run`, `pnpm storybook:start-design-system`. Per package: `pnpm --filter <name> run <script>`.

---

## Code Style & Formatting

### Prettier

Config: `@talend/scripts-config-prettier` (see `tools/scripts-config-prettier/.prettierrc.js`).

| Setting         | Value            |
| --------------- | ---------------- |
| Print width     | 100              |
| Quotes          | Single (`'`)     |
| Trailing commas | All              |
| Semicolons      | Yes              |
| Indentation     | **Tabs**         |
| Arrow parens    | Avoid (`x => x`) |
| JSON / rc files | 2-space indent   |
| SCSS files      | 1000 print width |

Prettier runs automatically on commit via `lint-staged` (Husky) on `*.{json,md,mdx,html,js,jsx,ts,tsx}`.

### EditorConfig

- LF line endings, UTF-8
- Trim trailing whitespace, insert final newline
- Tabs for `.js`, `.jsx`, `.css`, `.scss`
- 2-space indent for `.json`

### Linting (oxlint)

Config: `@talend/scripts-config-oxlint` (`tools/scripts-config-oxlint/index.mjs`), consumed by each package's `oxlint.config.mts` (and the root one).

- Run: `pnpm oxlint:run` (all) or `oxlint` inside a package
- **No `console.log`** — only `console.warn` and `console.error` allowed
- JSX only in `.jsx` / `.tsx` files
- Prefer named exports
- Avoid `any` in `.ts`/`.tsx` (warning)
- Follow `react-hooks` rules (`rules-of-hooks` error, `exhaustive-deps` warning) and `jsx-a11y`

`tools/scripts-config-eslint` and `tools/eslint-plugin` still exist in the repo, but oxlint is the linter used by the workspace scripts.

### Stylelint

Config: `stylelint-config-sass-guidelines` (see `tools/scripts-config-stylelint/.stylelintrc.js`).

- Tab indentation
- No `!important` (`declaration-no-important`)
- No `transition: all` — be specific about transitioned properties
- Max nesting depth: 5
- Lowercase hex colors, named colors where possible
- No unspaced `calc()` operators

---

## TypeScript

Base config: `@talend/scripts-config-typescript/tsconfig.json` (see `tools/scripts-config-typescript/`).

| Setting                            | Value       |
| ---------------------------------- | ----------- |
| `strict`                           | `true`      |
| `target`                           | `ES2015`    |
| `module`                           | `esnext`    |
| `moduleResolution`                 | `bundler`   |
| `jsx`                              | `react-jsx` |
| `declaration`                      | `true`      |
| `sourceMap`                        | `true`      |
| `isolatedModules`                  | `true`      |
| `esModuleInterop`                  | `true`      |
| `forceConsistentCasingInFileNames` | `true`      |
| `skipLibCheck`                     | `true`      |

Each package has a local `tsconfig.json` that extends this base:

```jsonc
{
	"extends": "@talend/scripts-config-typescript/tsconfig.json",
	"include": ["src/**/*"],
	"compilerOptions": {
		"rootDirs": ["src"],
	},
}
```

---

## Component Architecture

### Closed API Pattern (Design System)

Design system components (`packages/design-system`) use **closed APIs** — consumers cannot pass `className`, `style`, or `css` props. This ensures visual homogeneity across all products.

- **Atoms** (Button, Link, Input): single-tag elements, accept `string` children, typed to mirror their HTML counterparts. Props extend native HTML attributes minus `className`/`style`.
- **Molecules/Organisms** (Modal, Dropdown, Combobox): assembled components with rich props-based APIs. No composition — consumers hydrate via typed props.
- **Templates/Layouts**: may use composition (`children`) for page-level arrangement.

### Styling

- **CSS Modules** with `.module.css` files — this is the standard for all new code. No Styled Components.
- **Design tokens** via CSS custom properties from `@talend/design-tokens`. Use them for all colors, spacing, fonts, border-radius, shadows, transitions, etc.
- Use the `classnames` library for conditional class merging.

### Component Conventions

- Support `ForwardRef` — wrap components with `forwardRef` so consumers can pass refs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Talend/ui](https://github.com/Talend/ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

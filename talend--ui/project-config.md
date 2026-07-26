---
trigger: always_on
description: This is **Talend/UI**, a Yarn workspaces monorepo containing shared front-end libraries for Talend products.
---

# Talend/UI — AI Coding Instructions

## Repository Overview

This is **Talend/UI**, a Yarn workspaces monorepo containing shared front-end libraries for Talend products.

- **Workspaces**: `packages/*`, `tools/*`, `fork/*`
- **Stack**: React 18, TypeScript 5, Babel 7
- **Build tooling**: shared `@talend/scripts-*` packages (see `tools/`)
- **Versioning**: [Changesets](https://github.com/changesets/changesets) (`@changesets/cli`)
- **Package manager**: Yarn 1 (classic)

Run `yarn install` at the root. The `postinstall` script builds all libraries (`build:lib` + `build:lib:esm`).

---

## Code Style & Formatting

### Prettier

Config: `@talend/scripts-config-prettier` (see `tools/scripts-config-prettier/.prettierrc.js`).

| Setting          | Value              |
| ---------------- | ------------------ |
| Print width      | 100                |
| Quotes           | Single (`'`)       |
| Trailing commas  | All                |
| Semicolons       | Yes                |
| Indentation      | **Tabs**           |
| Arrow parens     | Avoid (`x => x`)   |
| JSON / rc files  | 2-space indent     |
| SCSS files       | 1000 print width   |

Prettier runs automatically on commit via `lint-staged` on `*.{json,md,mdx,html,js,jsx,ts,tsx}`.

### EditorConfig

- LF line endings, UTF-8
- Trim trailing whitespace, insert final newline
- Tabs for `.js`, `.jsx`, `.css`, `.scss`
- 2-space indent for `.json`

### ESLint

Each package has an `.eslintrc.json` extending `@talend` (resolved from `@talend/eslint-config` → `tools/scripts-config-eslint`).

Key rules and extends:

- `eslint:recommended`, `airbnb-base`, `plugin:prettier/recommended`
- `plugin:react/recommended`, `plugin:react/jsx-runtime`
- `plugin:react-hooks/recommended` — `rules-of-hooks` is error, `exhaustive-deps` is warning
- `plugin:jsx-a11y/recommended`
- `plugin:testing-library/react`, `plugin:jest-dom/recommended`
- `plugin:storybook/recommended`

Important rules:

- **No `console.log`** — only `console.warn` and `console.error` allowed
- JSX only in `.jsx` / `.tsx` files (`react/jsx-filename-extension`)
- `@talend/import-depth` (error) — controls import depth into packages
- `import/prefer-default-export`: off — named exports are fine
- `react/jsx-props-no-spreading`: off — spread is allowed
- `react/require-default-props`: off
- `@typescript-eslint/no-explicit-any`: warning (not error) in `.ts`/`.tsx` files
- `import/no-extraneous-dependencies`: off in test and story files

For TypeScript projects, the config auto-detects `tsconfig.json` and adds `@typescript-eslint` with `airbnb-typescript`.

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

| Setting                      | Value      |
| ---------------------------- | ---------- |
| `strict`                     | `true`     |
| `target`                     | `ES2015`   |
| `module`                     | `esnext`   |
| `moduleResolution`           | `bundler`  |
| `jsx`                        | `react-jsx`|
| `declaration`                | `true`     |
| `sourceMap`                  | `true`     |
| `isolatedModules`            | `true`     |
| `esModuleInterop`            | `true`     |
| `forceConsistentCasingInFileNames` | `true` |
| `skipLibCheck`               | `true`     |

Each package has a local `tsconfig.json` that extends this base:

```jsonc
{
  "extends": "@talend/scripts-config-typescript/tsconfig.json",
  "include": ["src/**/*"],
  "compilerOptions": {
    "rootDirs": ["src"]
  }
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
- Match native HTML element types — component props should extend the underlying element's attributes (e.g., `HTMLButtonElement` for buttons).
- Export components from the package's root `index.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Talend/ui](https://github.com/Talend/ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

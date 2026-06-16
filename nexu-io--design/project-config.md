---
trigger: always_on
description: - Applies to the entire repository rooted at `/Users/mrc/Projects/nexu-io/design`.
---

# AGENTS.md

## Scope
- Applies to the entire repository rooted at `/Users/mrc/Projects/nexu-io/design`.
- This is a pnpm workspace for a React/TypeScript UI library plus Storybook.
- This repo does not use `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md`; do not add them without discussion.

## Document map
- Design-system, UI, component usage, and token rules: `docs/design-system-guidelines.md`
- Product copy and localization policy: `docs/copy-and-localization.md`
- Shared component API design guidance: `docs/component-api-guidelines.md`
- Release workflow: `docs/release-flow.md`
- Package publishing and local consumption: `docs/package-publishing-and-consumption.md`
- Component API reference: `packages/ui-web/COMPONENT_REFERENCE.md`

## Repository map
- `package.json` — root workspace scripts.
- `pnpm-workspace.yaml` — workspace packages: `apps/*`, `packages/*`.
- `biome.json` — formatting rules.
- `tsconfig.base.json` — shared strict TypeScript settings.
- `packages/ui-web` — main React component library.
- `packages/tokens` — shared token package and CSS.
- `apps/storybook` — local component playground and docs.

## Package manager
- Use `pnpm` only.
- Workspace package names:
  - `@nexu-design/ui-web`
  - `@nexu-design/tokens`
  - `@nexu-design/storybook`

## Install
- `pnpm install`

## Common root commands
- Start Storybook: `pnpm dev`
- Build Storybook: `pnpm build`
- Build publishable packages: `pnpm build:packages`
- Create a changeset entry: `pnpm changeset`
- Format everything: `pnpm format`
- Check formatting: `pnpm format:check`
- Run Biome checks: `pnpm biome:check`
- Run all tests in workspace: `pnpm test`
- Run all type checks: `pnpm typecheck`
- Run all package lint scripts: `pnpm lint`
- Apply pending version bumps: `pnpm version:packages`
- Publish pending releases: `pnpm release`
- Run release readiness checks: `pnpm release:check`

## Package-specific commands

### `packages/ui-web`
- Build: `pnpm --filter @nexu-design/ui-web build`
- Test all: `pnpm --filter @nexu-design/ui-web test`
- Typecheck: `pnpm --filter @nexu-design/ui-web typecheck`
- Lint: `pnpm --filter @nexu-design/ui-web lint`
- Pack dry run: `pnpm --filter @nexu-design/ui-web pack:check`

### `packages/tokens`
- Build: `pnpm --filter @nexu-design/tokens build`
- Typecheck: `pnpm --filter @nexu-design/tokens typecheck`
- Lint: `pnpm --filter @nexu-design/tokens lint`
- Pack dry run: `pnpm --filter @nexu-design/tokens pack:check`

### `apps/storybook`
- Start: `pnpm --filter @nexu-design/storybook storybook`
- Build: `pnpm --filter @nexu-design/storybook build-storybook`
- Typecheck: `pnpm --filter @nexu-design/storybook typecheck`
- Lint: `pnpm --filter @nexu-design/storybook lint`

## Formatting and TypeScript rules
- Formatter is Biome (`biome.json`).
- Indentation: 2 spaces.
- Max line width: 100.
- JavaScript/TypeScript quotes: single quotes.
- JSX attribute/string quotes: double quotes.
- Semicolons: as needed, not mandatory.
- Trailing commas: ES5 style.
- Respect existing file formatting; do not introduce a conflicting style.
- The repo uses strict TypeScript (`strict: true`).
- Target/runtime baseline: ES2022 modules with bundler resolution.
- Keep `noEmit` expectations for typecheck commands.
- Prefer explicit prop types for public components.
- Reuse platform types like `React.ButtonHTMLAttributes<HTMLButtonElement>`.
- Use `VariantProps<typeof ...>` when a component is driven by CVA variants.
- Preserve declaration-output compatibility for package builds.
- Avoid `any`; use precise types, unions, generics, or `unknown` plus narrowing.

## Naming, imports, and component patterns
- Source filenames are kebab-case: `button.tsx`, `form-field.tsx`, `radio-group.tsx`.
- Test filenames mirror source names: `button.test.tsx`.
- Storybook stories use `*.stories.tsx`.
- Exported React component names are PascalCase.
- Utility functions use camelCase.
- Props interfaces are PascalCase with a `Props` suffix.
- Context value interfaces use descriptive names.
- Follow the existing import grouping pattern:
  1. React import(s)
  2. third-party packages
  3. blank line
  4. local relative imports
- Prefer named exports over default exports.
- Keep barrel exports updated in `packages/ui-web/src/index.ts` when adding public API.
- Use `import type` where only types are needed when it improves clarity.
- Components commonly use `React.forwardRef` for DOM-facing primitives.
- Set `displayName` on `forwardRef` components.
- Prefer composition via Radix primitives rather than custom low-level behavior.
- Support `className` merging via the shared `cn()` helper.
- Use CVA (`class-variance-authority`) for variant-driven styling.
- Keep accessibility props and roles intact when wrapping Radix or native elements.
- When using `asChild`, preserve disabled/loading behavior carefully.
- Before changing public UI, visual behavior, layout, variants, or interaction patterns, read `docs/design-system-guidelines.md`.
- Before changing product-surface copy or i18n behavior, read `docs/copy-and-localization.md`.

## Tests and accessibility
- Test runner: Vitest.
- Config: `packages/ui-web/vitest.config.ts`.
- Test environment: `jsdom`.
- Setup file: `packages/ui-web/src/test/setup.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nexu-io/design](https://github.com/nexu-io/design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

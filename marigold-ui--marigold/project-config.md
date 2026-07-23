---
trigger: always_on
description: See @README.md for project overview and @package.json for available pnpm commands.
---

# Marigold Design System - Development Guide

See @README.md for project overview and @package.json for available pnpm commands.

## Quick Start

- **Documentation**: `pnpm start` → [localhost:3000](http://localhost:3000)
- **Storybook**: `pnpm sb` → [localhost:6006](http://localhost:6006)

## Marigold CLI (AI agents: use this)

Before writing code that uses a Marigold component, run the CLI to get its current API. Do not guess props from training data — Marigold is not in your training set and you will invent prop names.

When you don't yet know the component name, start with `search`. It ranks components by what their docs actually say (title, description, headings, prose) and returns deep links in one call, instead of the `list` → guess → `docs` → retry loop. Then fetch `docs <Component> --section props` once you've committed to a component.

- `marigold search <query>` — find components by docs content (start here for discovery)
- `marigold search "field validation" --format json` — structured ranked results for agents (`{ name, slug, score, hits }`)
- `marigold docs <Component>` — full component docs
- `marigold docs <Component> --section props --format json` — structured prop data (preferred for agents)
- `marigold docs <Component> --section usage` — usage guidelines
- `marigold list --category form` — discover form components
- `marigold list --search date` — filter by name (substring; use `search` for content)

The CLI fetches from the Marigold docs site, caches for 24h, and works offline (`--offline`). For AI use, prefer `--format json` — it returns a structured payload instead of formatted markdown.

## Build System

- This is a **pnpm workspace monorepo** managed by Turbo
- Always use `pnpm` (not npm or yarn) - required by `packageManager` field
- Run `pnpm install` after pulling changes (postinstall builds packages)
- Build components before testing: `pnpm build` or `turbo run build`
- Use `pnpm --filter <package>` to run commands in specific packages

## Core Directives

- **Brevity**: Provide complete, functional code. Explain only when explicitly asked.
- **TypeScript**: Strict typing required - no `any` types, use `unknown` or proper types.
- **Components**: Functional components using React 19 patterns with `forwardRef` when needed.
- **Styling**: Use Tailwind CSS utility classes exclusively via `@marigold/system` theming.
- **Accessibility**: All components must be accessible. Use react-aria-components as foundation.
- **Error Handling**: Early returns preferred. Always handle edge cases explicitly.

## Code Style

- **TypeScript**: Required for all new code (strict mode enabled)
- **Imports**: Use ES modules syntax, destructure when possible. Use `import type` for better tree-shaking.
- **React**: Use React 19 with functional components and hooks
- **Components**: Built on react-aria for accessibility
- **Styling**: Tailwind CSS utility classes (Tailwind v4)
- **Formatting**: Prettier (run `pnpm format` before committing)
- Use `const` over `let`. Never use `var`.
- Name component files in `PascalCase` (e.g., `Button.tsx`)
- Name utility files in `camelCase` (e.g., `useNonModal.ts`)
- Use the `useClassNames` hook from `@marigold/system` for theming
- Rename react-aria props: `isDisabled` → `disabled`, `isPending` → `loading`
- Export components with named exports
- Use React Context for component composition (see `AccordionContext` patterns)

## Testing

- **Test runner**: Vitest (not Jest)
- **Run tests**: `pnpm test` (all), `pnpm test:unit` (unit tests), `pnpm test:sb` (story tests)
- **Story tests**: Add `play` functions with `tags: ['component-test']` - runs in real browser via Playwright
- **Unit tests**: Import stories (e.g., `<Basic.Component />`) instead of creating test fixtures
- **Browser tests**: Playwright integration via `@vitest/browser-playwright`
- **Coverage**: `pnpm test:coverage` (requires 90% statements, functions, lines; 85% branches)

## Workflow

- **Typecheck**: Run `pnpm typecheck:only` after code changes
- **Lint**: Run `pnpm lint` to check code style
- **Format**: Run `pnpm format` before committing
- **Branch from**: `main` (use GitHub Flow)
- **Changesets**: Use `pnpm changeset` for version management
- **Storybook**: Run `pnpm sb` to preview components locally

## Monorepo Structure

- `packages/components` - Core React components
- `packages/system` - Design system utilities and hooks
- `packages/icons` - Icon components
- `packages/types` - Shared TypeScript types
- `themes/*` - Theme packages (theme-rui)
- `docs` - Documentation site (Next.js)
- `.storybook` - Storybook configuration

## Do Not

### Testing

- **Don't create test fixtures/themes** - Import stories instead of using `setup()` or creating inline themes
- **Never use `fireEvent`** - Always use `userEvent` (from `storybook/test` in stories, or `@testing-library/user-event` in test files)
- **Never use Jest APIs** - Use Vitest equivalents (`vi.fn()`, `vi.spyOn()`, `vi.mock()` instead of `jest.*`)
- **Avoid `getByTestId`** - Prefer accessible queries: `getByRole`, `getByLabelText`, `getByText` (Testing Library best practice)
- **Don't wrap in `act()` unnecessarily** - `userEvent` and `render` already handle this

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marigold-ui/marigold](https://github.com/marigold-ui/marigold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

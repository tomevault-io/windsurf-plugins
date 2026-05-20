---
trigger: always_on
description: Favor readable, skimmable, well-verified code over speed or cleverness.
---

# Nakafa Codebase Agent Guide

Build for longevity.
Favor readable, skimmable, well-verified code over speed or cleverness.

## Source Of Truth

- Use retrieval-led reasoning first. Read the repo, configs, docs, and generated files before changing code.
- No Cursor rules were found in `.cursor/rules/` or `.cursorrules` when this guide was updated.
- No Copilot instructions were found in `.github/copilot-instructions.md` when this guide was updated.
- This root `AGENTS.md` is the baseline for the repo.
- Also read any nested `AGENTS.md` files in the area you touch.
- For Convex work, `packages/backend/AGENTS.md` is mandatory.
- For Convex work, read `packages/backend/convex/_generated/ai/guidelines.md` before making changes.
- For structural work, skim recent `git log` first so you do not reintroduce old patterns.
- Verify behavior from actual code and docs, not memory.

## Stack And Layout

- Package manager: `pnpm@10.33.0`
- Runtime: Node `>=22`
- Monorepo: Turborepo
- Frontend: Next.js 16, React 19, TypeScript 6
- Backend: Convex
- Lint/format: Biome via Ultracite
- Tests: Vitest
- Apps: `apps/www`, `apps/api`, `apps/mcp`, `apps/email`
- Main packages: `packages/backend`, `packages/design-system`, `packages/contents`, `packages/ai`, `packages/testing`
- App-local alias: `@/*`
- Cross-package alias: `@repo/*`
- Educational MDX content lives in `packages/contents/`

## Required Reading

- Convex best practices: `https://docs.convex.dev/understanding/best-practices/`
- Convex Workflow: `https://www.convex.dev/components/workflow`
- Convex cron jobs: `https://docs.convex.dev/scheduling/cron-jobs`
- Functional relationships helpers: `https://stack.convex.dev/functional-relationships-helpers`
- Convex Helpers README: `https://github.com/get-convex/convex-helpers/blob/main/packages/convex-helpers/README.md`
- Convex Aggregate: `https://www.convex.dev/components/aggregate`
- Convex pagination: `https://docs.convex.dev/database/pagination`
- React effects guidance: `https://react.dev/learn/you-might-not-need-an-effect`
- Mantine Hooks docs: read before building new hook utilities.
- Read installed Convex node modules when behavior matters.
- Read Convex MCP code/docs when the change touches MCP or agent tooling.
- Use Context7 for up-to-date library docs and Ultracite search for lint/style rules.
- 
## Core Commands
- `pnpm dev` - run the main web app and Convex backend through Turbo
- `pnpm dev:web` - run the main web app and Convex backend through Turbo
- `pnpm dev:all` - run all workspace `dev` tasks through Turbo
- `pnpm build` - build all packages and apps
- `pnpm test` - run all workspace tests
- `pnpm test:watch` - run watch-mode tests where supported
- `pnpm test:ui` - open Vitest UI where supported
- `pnpm test:coverage` - run coverage across workspaces that support it
- `pnpm lint` - run `ultracite check`
- `pnpm format` - run `ultracite fix`
- `pnpm format:doctor` - run `ultracite doctor`
- `pnpm analyze` - run analyze tasks
- `pnpm boundaries` - run workspace boundaries checks

## Focused Commands

- `pnpm --filter www dev` - Next.js app on `3000`
- `pnpm --filter api dev` - Next.js API app on `3002`
- `pnpm --filter mcp dev` - MCP app on `3001`
- `pnpm --filter email dev` - React Email preview on `3004`
- `pnpm --filter @repo/backend dev` - Convex dev with tailed logs
- `pnpm dev` / `pnpm dev:web` - recommended day-to-day combo for `www` + Convex backend
- `pnpm dev:all` - use when you truly need every app running together
- There is no single root `typecheck` script; run it in the workspace you changed.
- `pnpm --filter www typecheck`
- `pnpm --filter api typecheck`
- `pnpm --filter @repo/backend typecheck`
- `pnpm --filter @repo/design-system typecheck`
- Preferred single-test pattern: `pnpm --filter <workspace> exec vitest run <relative-test-path>`
- `pnpm --filter www exec vitest run __tests__/pages.test.tsx`
- `pnpm --filter api exec vitest run lib/__tests__/proxy.test.ts`
- `pnpm --filter @repo/backend exec vitest run helpers/chunk.test.ts`
- `pnpm --filter @repo/contents exec vitest run _lib/__tests__/content.test.ts`
- Add `-t "test name"` to run one named test.
- Use `pnpm --filter <workspace> test` for the whole workspace suite.
- Convex helpers: `pnpm --filter @repo/backend setup`, `seed`, `deploy`, `insights`

## Workflow Expectations

- Read the touched package's `package.json`, config files, and nearby code before editing.
- Follow existing local patterns before inventing new abstractions.
- Keep changes cohesive across frontend, Convex backend, and dev/prod behavior.
- Avoid disconnected patches, workaround code, wrapper chains, and legacy leftovers.
- Prefer small, direct helpers over abstraction layers.
- Use early returns to keep logic flat and skimmable.
- Run the smallest useful verification set after changes, then expand if risk is high.

## Formatting And Imports

- Formatting is owned by Biome through Ultracite. Run `pnpm format` instead of hand-formatting.
- Use spaces for indentation.
- Use double quotes for strings and JSX attributes.
- Prefer `import type` for type-only imports.
- Within apps, prefer `@/` imports over long relative paths.
- Across workspaces, use `@repo/*` imports.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nakafaai/nakafa.com](https://github.com/nakafaai/nakafa.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

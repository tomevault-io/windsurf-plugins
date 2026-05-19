---
trigger: always_on
description: `AGENTS.md` is the repo-local source of truth for agent guidance. Do not add parallel tool-specific guidance files such as `CLAUDE.md`; keep shared instructions here so they stay current for every coding agent.
---

# Repository Guidelines

## Guidance Source
`AGENTS.md` is the repo-local source of truth for agent guidance. Do not add parallel tool-specific guidance files such as `CLAUDE.md`; keep shared instructions here so they stay current for every coding agent.

## Plans
When I ask you to execute a plan file, after completing each step: check it off in the plan and immediately make a commit for that step. The first commit should add the plan, and each additional commit should check off one step.

## Project Structure & Module Organization
Legend List is a TypeScript React Native and React DOM list library. Core source lives in `src/`, split into feature folders such as `components` (public list primitives and rendering containers), `core` (scroll orchestration, layout, and viewability logic), `state` (shared state and context machinery), `hooks`, `utils`, `platform`, `integrations`, and `section-list`. Shared base types live in `src/types.base.ts`, with public entrypoint wrappers in `src/types.web.ts`, `src/types.react-native.ts`, and `src/types.root.ts`. Unit and regression tests mirror this layout in `__tests__/`. `example/` contains the Expo React Native showcase app, and `example-web/` houses the Vite playground used for web fixtures and comparisons. Built artifacts land in `dist/` after a release and should not be edited manually.

## Build, Test, and Development Commands
Install dependencies with `bun install`. Use `bun run build` to generate production bundles via `tsup` followed by post-build cleanups. Run `bun test` for the full suite, `bun test --watch` during local iteration, and `bun test --coverage` before publishing. `bun run lint` executes Biome checks across `src`, `__tests__`, `example/app`, `example/screens`, and `example-web`. Type-level regressions are caught with `bun run tsc`; use `bun run tsc:src`, `bun run tsc:example`, or `bun run tsc:example-web` for narrower checks. `bun run tsc:go` is available when using ts-go-to-definition tooling.

Example app commands live in their own packages. For native scenarios, use `cd example && bun run ios`, `bun run android`, or the `*:fixtures` variants. For web scenarios, use `cd example-web && bun run dev` or `bun run dev:fixtures`.

## Architecture Notes
`LegendList` wraps the public component in a state provider and coordinates virtualization, scroll state, anchor tracking, and item positioning. The container system in `src/components/Container*.tsx` recycles rendered item containers when `recycleItems` is enabled; treat local state inside recycled item components carefully because the same mounted container can represent different items over time.

Dynamic item sizing is estimate-first, then measurement-corrected. Prefer `estimatedItemSize` for initial size hints, or `getFixedItemSize` when item sizes are known exactly. `getEstimatedItemSize` remains supported only as a deprecated per-item estimate escape hatch. Size changes feed back into total size, item positions, viewability, and maintain-visible-content-position behavior, so layout changes should include tests around scroll offsets and anchor stability.

The scroll adjustment paths are intentionally sensitive. Changes touching `bootstrapInitialScroll`, `maintainScrollAtEnd`, `maintainVisibleContentPosition`, item measurement, container allocation, or viewability should include focused regression tests and, when useful, an `example-web` fixture that exercises the visible behavior.

## Coding Style & Naming Conventions
We rely on Biome for formatting; always run the lint script before pushing to ensure consistent 4-space indentation, trailing commas, and sorted imports. Components and classes use PascalCase, hooks use the `useFeature` camelCase pattern, and helper functions stay camelCase. Constants defined in `constants.ts` and similar modules use SCREAMING_SNAKE_CASE. Prefer explicit exports through barrel files (`src/index.ts`) to keep the public surface predictable.

## Lint / Formatting
- For any task that changes code, do not call the work done until formatting and lint autofix has been handled.
- Prefer writing changes in Biome-conforming style from the start, but still run `bun run lint:fix` before declaring completion when it is safe to do so.
- If repo-wide autofix would create unrelated churn in a dirty worktree, limit fixes to the touched files when possible; otherwise explicitly report that lint autofix was not run and why.
- If unused-import cleanup requires it, use `bun run lint:fix-unsafe` deliberately rather than leaving that cleanup for the user.

## Testing Guidelines
Unit tests live alongside features inside `__tests__` and should mirror file names from `src` with a `.test.tsx` suffix. Use React Native Testing Library and the helpers in `test-utils` for rendering scenarios; avoid snapshot churn by relying on semantic assertions. New behavior must include coverage and update related visualization artifacts via `bun run test:visualize` when debugging scroll offsets.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LegendApp/legend-list](https://github.com/LegendApp/legend-list) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

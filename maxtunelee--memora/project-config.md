---
trigger: always_on
description: - Workspace managed with `pnpm` (see `pnpm-workspace.yaml`).
---

# AGENTS.md

## Repository overview

- Workspace managed with `pnpm` (see `pnpm-workspace.yaml`).
- Primary package: `@memora/web` in `packages/web`.
- Frontend stack: React 19, Vite (rolldown-vite), Tailwind CSS v4.
- Routing uses `react-router` plus `vite-plugin-route-builder`.

## Language and interface copy

- Use direct, natural language. Avoid inflated wording and development slang in user-facing communication.
- Do not write redundant eyebrow labels. Remove them instead of changing them to sentence case.
- For necessary interface labels, use sentence case. Do not use all-uppercase section labels, metric labels, status chips, tabs, buttons, or navigation items, and do not use letter-spaced uppercase text to create hierarchy.
- Preserve canonical capitalization for technical acronyms such as OCR, ASR, LLM, OPFS, and API.

## Commands (build/lint/test)

### Workspace basics

- Install deps: `pnpm install`
- Run commands for a package: `vp run @memora/web#<command>`
- Run the web app with workspace dependencies: `vp run -t @memora/web#dev`
- Run the marketing site (`@memora/site`, port `9004`): `vp run @memora/site#dev`
  - It renders real `@memora/web` components from source on mock data; `@/livestore/store` is aliased to a stub, so only embed components that don't need the store.
- Build the web app with workspace dependencies: `vp run -t @memora/web#build`

### Development

- Dev server with workspace dependencies: `vp run -t @memora/web#dev`
  - Vite runs on port `9003` (see `packages/web/vite.config.ts`).
- Preview build: `vp --filter @memora/web preview`

### Build

- Build web app and declared workspace dependencies: `vp run -t @memora/web#build`
- Vite Task follows `workspace:*` dependencies before running the web build.

### Lint

- Lint web app: `pnpm --filter @memora/web lint`
  - Oxlint configured for TypeScript and React rules via `packages/web/.oxlintrc.json`.

### Format

- Format workspace: `vp fmt .`
- Check formatting: `vp fmt . --check`
  - Oxfmt reads config from the `fmt` block in the root `vite.config.ts`.

### Tests

- Web app tests live under `packages/web/test/<module>/...`.
- Run all web tests: `pnpm --filter @memora/web test`
- Run a single web test file: `pnpm --filter @memora/web test -- test/<module>/<file>.test.ts`

## Code style and conventions

### Formatting

- Use double quotes for strings.
- Use semicolons.
- Use trailing commas where Oxfmt would allow.
- Keep import blocks separated from other code with a blank line.
- Keep JSX wrapped with parentheses and multiline formatting.

### Imports

- Prefer ES module imports.
- Group imports by source:
  1. React/third-party
  2. Internal modules (`./`, `../`)
- Use named imports when possible; default imports for components.

### TypeScript

- `strict: true` and unused checks enabled.
- Avoid `any`; use concrete types or generics.
- Use `type` for unions/intersections, `interface` for object shapes.
- Favor explicit return types for exported functions.
- Use `as const` when creating literal unions.
- Avoid non-null assertions unless a runtime guarantee exists.

### React

- Function components only.
- Keep hooks at the top-level of components.
- Prefer memoized callbacks with `useCallback` when passing down.
- Prefer `useMemo` only for expensive computations.
- Use `StrictMode` (already set in `packages/web/src/main.tsx`).

### Naming

- `camelCase` for variables/functions.
- `PascalCase` for React components/types/interfaces.
- Constants in `SCREAMING_SNAKE_CASE` when truly constant.
- Files: `camelCase.ts` for utilities, `PascalCase.tsx` for components/pages.

### Error handling

- Handle async errors explicitly; fail fast where appropriate.
- Prefer early returns over deep nesting.
- For user-facing errors, return safe defaults or fallbacks.
- Validate external inputs before use (e.g., `Number.isFinite`).

### State and side effects

- Keep derived state out of `useState` when possible.
- Clean up side effects in `useEffect` return functions.
- Avoid mutations of React state arrays/objects.

### CSS/Tailwind

- Use Tailwind utility classes and `tailwind-merge` helpers where needed.
- Reuse shared class strings via helpers in `src/lib` if repeated.

## Releases and versions

- The app version is the build's git short hash (`git describe --always --dirty`), shown in Settings > About.
- For a change people will notice, add a bullet under a new top `## <release date>` section in `packages/web/RELEASE_NOTES.md`, in plain user-facing language. The update dialog shows the sections a user has not seen yet.
- The service worker waits for the user to accept an update from that dialog; it does not activate new builds on its own.
- Third-party runtime files copied into the build (ONNX Runtime, VAD, sqlite-vec) are served from `/vendor/<package>@<version>/`. Reference them through `__VENDOR_ASSETS__`, not fixed paths.

## Generated files

- `packages/web/src/generated-routes.ts` is generated by the route builder.
- Do not edit generated files directly.

## Vite and build notes

- Vite uses React Compiler via `babel-plugin-react-compiler`.
- Static assets (VAD worklets, ONNX, WASM) are copied via `vite-plugin-static-copy`.
- Build targets `esnext` with sourcemaps enabled.

## Workspace instructions (from .github/instructions)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaxtuneLee/memora](https://github.com/MaxtuneLee/memora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

---
trigger: always_on
description: GlowTour.js is a production cross-framework guided-tour library managed as a Bun workspace monorepo. The public packages are `packages/core`, `packages/react`, `packages/vue`, `packages/angular`, `packages/solid`, `packages/vanilla`, and `packages/styles`. Reference applications live in `apps/`: `playground`, `website`, `ssr-react`, `ssr-vue`, `ssr-solid`, `ssr-angular`, and `screen-readers` (assistive-technology tests run in CI).
---

# AGENTS.md

GlowTour.js is a production cross-framework guided-tour library managed as a Bun workspace monorepo. The public packages are `packages/core`, `packages/react`, `packages/vue`, `packages/angular`, `packages/solid`, `packages/vanilla`, and `packages/styles`. Reference applications live in `apps/`: `playground`, `website`, `ssr-react`, `ssr-vue`, `ssr-solid`, `ssr-angular`, and `screen-readers` (assistive-technology tests run in CI).

These instructions apply to any coding agent working in this repository.

## Project boundaries

- `packages/core` owns workflow state, positioning, DOM behavior, and framework-agnostic primitives. It must not own presentation or hardcode browser globals such as `window`, `document`, `location`, or storage APIs in code paths that would break SSR.
- Framework-specific rendering and lifecycle integration belong in the corresponding adapter package.
- `packages/styles` owns the default CSS theme and design tokens.
- Keep behavior consistent across adapters unless a framework requires a documented exception.
- Public API compatibility matters: GlowTour.js is already published and used as a library. Treat public API changes as potentially breaking.
- npm publication happens through GitHub Actions. Never publish packages manually.

## Before changing code

- Read the relevant package, its tests, and nearby conventions before editing.
- Prefer the smallest change that solves the actual problem. Do not introduce speculative abstractions or unrelated refactors.
- Reuse existing primitives and contracts before adding new public API.
- If behavior, public API, SSR semantics, accessibility, or package exports could change, identify that impact explicitly before implementation.

## API design: recipe before feature

The project favors flexibility and DX by keeping the public API small.

- Before adding public API, verify that consumers cannot already express the behavior from existing primitives in a few lines. If they can, document a recipe instead of adding another option.
- Core should expose primitives; application policy stays in the application. Storage, routing, analytics, i18n, TTL, and multi-tab policy should not be absorbed into core without strong repeated evidence.
- Do not provide two public ways to solve the same problem without a compelling reason.
- Prefer a clear required contract over optional fields that create parallel behavior modes.
- Adding API later is additive; removing API later is breaking. When uncertain, prefer not to expand the public surface.
- Keep documentation claims strictly aligned with behavior that the library actually implements and tests.
- Record important rejected API directions and their rationale in `docs/` so the same decision does not have to be rediscovered later.

## Implementation rules

- Preserve TypeScript type safety and explicit boundaries.
- Keep tree-shaking in mind when changing exports or package entry points.
- Preserve accessibility behavior: focus management, keyboard handling, ARIA semantics, and restoration are product requirements, not optional polish.
- Do not introduce runtime dependencies casually. The core is intentionally dependency-free at runtime.
- Do not hide browser-only behavior behind types alone. Validate actual runtime behavior.
- Changes that affect users or published packages need an appropriate Changeset unless the repository conventions clearly classify them as documentation/internal-only work.

## Required validation

Validation is part of the implementation. Do not report a package change as complete until the relevant checks have actually passed.

### Automated checks

For package changes, run the complete repository validation unless the task is explicitly limited to documentation or non-runtime repository metadata:

```bash
bun install --frozen-lockfile
bun run check
bun run typecheck
bun run build
bun test
bun run test:browser
bun run pack
bun run test:tarballs
bun run --cwd apps/playground build
bun run --cwd apps/website build
```

Run targeted tests while iterating, but targeted tests do not replace the full relevant validation before completion. Never claim that tests pass from inspection alone; use fresh command output.

### Browser validation: mandatory for package changes

Any behavioral modification under `packages/` must be exercised in a real browser. `apps/playground` is the reference application for package behavior.

- Start it with `bun run playground`.
- Exercise the changed behavior through the relevant framework example in the browser.
- Check the user-visible result, interactions, focus/keyboard behavior when relevant, and browser console errors.
- Do not consider unit tests, type checking, or a successful build a substitute for this browser validation.
- If browser validation cannot be performed in the current environment, state that explicitly and do not describe the change as fully verified.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Glowhop/GlowTour.js](https://github.com/Glowhop/GlowTour.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->

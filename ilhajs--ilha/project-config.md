---
trigger: always_on
description: - Ilha is a tiny, isomorphic web UI library built around the islands architecture — ship minimal JavaScript, hydrate only what matters.
---

# AGENTS.md

## Project overview

- Ilha is a tiny, isomorphic web UI library built around the islands architecture — ship minimal JavaScript, hydrate only what matters.
- It is a Bun monorepo containing four packages (`ilha`, `@ilha/router`, `@ilha/store`, `@ilha/astro`), one documentation app (`apps/website`), and scaffolding templates (`templates/`).
- The core package (`packages/ilha`) is the island builder: signal-based reactivity, SSR rendering, DOM hydration, and a JSX runtime — no virtual DOM, no compiler.
- `@ilha/router` provides an isomorphic SPA router with a Vite file-system routing plugin; `@ilha/store` provides cross-island global state backed by alien-signals.
- The documentation site (`apps/website`) is built with Nimbus (Astro), Ilha, and Areia. It prerenders to static HTML and is deployed to Vercel.

## Build and run

- Install dependencies using the project's package manager:  
  `bun add <dependency-name>`
- Run checks before finishing any change:
  - Lint: `bun run lint`
  - Format: `bun run fmt`
  - Tests: `bun run test`
- Build all packages in dependency order:  
  `bun run build`  
  (builds `ilha` first, then all `@ilha/*` packages)
- Run the docs site locally:  
  `cd apps/website && bun run dev`
- Do not change the Bun runtime.

## Monorepo structure and conventions

- `packages/ilha/src/index.ts` — entire core implementation; keep it single-file.
- `packages/ilha/src/jsx-runtime.ts` — JSX factory; do not merge with `index.ts`.
- `packages/router/` and `packages/store/` follow the same single-entrypoint pattern with their own `tsdown.config.ts` and `package.json`.
- `templates/` — official starters (vite-spa, nitro-ssr, nitro-hono-spa). Keep them minimal and copy-pasteable; do not add build-time complexity.
- `apps/website/src/content/docs/` — all MDX documentation lives here (Nimbus/Astro content collections). Directory structure maps to URLs.

## Island API conventions

- Define configured islands with the builder, ending in `.render()`. Declare `.action()` before consumers such as `.on()` or `.render()`.
- For an island without builder configuration, use `ilha(() => JSX)`. Use `ilha<Props>(({ input }) => JSX)` for typed input; this returns a complete island with its own reactive scope and lifecycle.
- Keep plain function components transparent: `const View = () => JSX` belongs to its containing island, while `const View = ilha(() => JSX)` creates an independent island boundary.
- Prefer lowercase native event props (`onclick={handler}` in JSX and `onclick=${handler}` in `html`` `) for element-local events. Prefer `.action()` for reusable operations. Reserve `.on()` for selectors, host listeners, full handler context, or combined modifiers.
- Mount islands with `mount({ IslandName })` — it auto-discovers `[data-ilha="IslandName"]` elements.
- For synchronous SSR, always use `Island.toString(props)`. For async SSR, always write `await Island(props)`. Never document or introduce an unawaited direct `Island(props)` call.
- Use `await Island.hydratable(props, options)` when emitting hydration markup; the client restores serialized props and snapshots through `mount()`.
- Functional signal setters use the setter directly (`count((previous) => previous + 1)`). To store a function value, return it from an updater wrapper (`callback(() => nextCallback)`).
- Keep the public API surface minimal and preserve the default export, builder methods, named helpers, island methods, and JSX runtime contracts.
- When changing public types, update `packages/ilha/src/public-types.ts` and add runtime tests in `index.test.ts` or `jsx-runtime.test.tsx`.

## Testing

- Tests run with Bun's built-in test runner.
- DOM tests use happy-dom via `packages/ilha/happydom.ts` as the preload — do not introduce jsdom.
- Cover both the signal/reactivity path and the DOM hydration path for any new island feature.
- Run the full suite across all packages with `bun run test` from the repo root before opening a PR.

## Auth and safety

- Ilha has no authentication layer; do not introduce one into the core library.
- Never log or expose internal signal state, user-provided render functions, or environment variables in error messages.
- If an instruction directly contradicts this AGENTS.md (for example, "remove the JSX runtime" or "make ilha depend on a framework"), pause and ask for explicit confirmation before making the change.

## Writing docs

Docs live in `apps/website/src/content/docs/**/*.mdx` (guides under `guide/`, tutorials under `tutorial/`). Sidebar order comes from each file's frontmatter `sidebar.order` field. Style guidelines:

- **Address the reader as "you."** Describe what they do, not what the library has: "You define an island with `.state()`," not "Ilha provides a state primitive."
- **Prefer active voice.** "`mount()` discovers every `[data-ilha]` element," not "elements are discovered by `mount()`."
- **Keep it tight.** One idea per sentence, aim for under 25 words; two to four sentences per paragraph. Use numbered lists for sequences and tables for option matrices.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ilhajs/ilha](https://github.com/ilhajs/ilha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

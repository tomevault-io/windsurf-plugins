---
trigger: always_on
description: transforms, inspired by Vega/Vega-Lite. GenomeSpy App is a higher-level
---

# GenomeSpy

GenomeSpy is a high-performance, web-based visual analytics toolkit for genomic
data. It combines a declarative visualization grammar with a custom GPU-
accelerated rendering engine to provide smooth interaction with large,
heterogeneous datasets, including copy number profiles, structural variants,
mutations, and metadata across cohorts. The project includes a modular core and
a schema-driven specification format.

GenomeSpy Core provides building blocks such as marks, scales, axes, and data
transforms, inspired by Vega/Vega-Lite. GenomeSpy App is a higher-level
interactive application for cohort analysis, built on the Core with provenance-
aware interactions.

## Tech stack in use

- Fully client-side application using modern web technologies
- JavaScript (Modern ESNext) typed with JSDoc
- TypeScript for more complex type definitions and JSON Schema generation
- Monorepo managed with lerna-lite

### Core

- WebGL rendering via twgl.js
- JSON Schema built from TypeScript types
- Application state is maintained in the view hierarchy
  - Data flow (built from FlowNode objects) handles data input and transformation
  - ScaleResolution collects views that share scales and initializes the scales
  - ParamMediator manages reactive parameters (signals)

### App

- Embeds GenomeSpy Core for rendering
- State management with Redux Toolkit (including provenance tracking)
- UI components with Lit
- Iconography with FontAwesome
- No external CSS frameworks or component libraries

### App AI Agent

- An LLM agent/chatbot is in early development, implemented in `packages/app-agent/` and the Python relay in `packages/app-agent/server/`.

### Testing

- Unit tests with Vitest
- Tests live next to code, with `.test.` in the filename
- When writing tests, add a short comment for non-obvious test setup/intent.
- Do not use TDD for trivial presentation-only changes such as changing a
  label, icon, tooltip, or other copy/style detail. Apply the small edit
  directly and verify with the lightest relevant check.
- Permanent tests should verify behavior, contracts, dataflow, layout
  semantics, or user-visible output rather than duplicating the current
  implementation.
- Prefer representative assertions that capture intent. Avoid exhaustive checks
  over every property of a configuration object, generated spec, or internal
  structure unless that full shape is an intentional compatibility contract.
- Transient tests that pin down implementation details are acceptable while
  debugging, but delete or rewrite them before committing. After a refactor,
  remove tests that only cover temporary compatibility paths or intermediate
  implementation details unless that behavior remains an intentional public
  contract.
- For generated specs, rendered hierarchy/layout inspection, or other structured
  output where the whole shape matters, prefer focused snapshot tests once the
  design has stabilized. Prefer `specToLayout(...)` or `renderToLayout(...)`
  from `packages/core/src/view/testUtils.js` instead of ad hoc scripts.
- If existing structures are unsuitable for snapshot testing, proactively
  propose a stable snapshot-friendly representation or test helper. Do not
  refactor production code or test infrastructure solely for snapshot testing
  without developer approval.
- `packages/core/layout.test.js` and `packages/core/src/view/layoutSnapshot.test.js` show the recommended layout-snapshot pattern.

### Running tests and linting

- From repo root, run the full unit suite: `npm test`
- Run a focused Vitest suite: `npx vitest run packages/app/src/sampleView/sampleView.test.js`
- TypeScript checks for workspaces (if present): `npm --workspaces run test:tsc --if-present`
- Lint the workspace sources: `npm run lint`

### Web App Debugging

- Prefer Vitest and Playwright. However,
- Use Chrome DevTools MCP (https://github.com/ChromeDevTools/chrome-devtools-mcp/) may also be available.
- Open `http://127.0.0.1:8080/` using Chrome DevTools MCP.
- If the dev server is not running, start it with `npm start` from the repo root.
- The root page lists example and private specs; the first example is usually the quickest smoke test.
- The dev routes live in the package `vite.config.js` files and share helper code from `devServerRoutes.mjs` at the repo root.

## Workflow expectations

- By default, make a plan first; don't start editing code when the user is asking
  a random question and hasn't explicitly asked for editing work to start.
- When planning, consider documentation in `docs/` for user-visible changes. New
  features in Core or App may need docs, while refactors typically do not.
- For refactors and simplification work, measure the relevant code size before
  and after the change, for example with `wc -l`, `git diff --stat`, or focused
  line counts. Treat added lines in a simplification task as a signal to re-check
  whether the result is actually simpler. More lines are acceptable only when
  they clearly improve correctness, readability, or maintainability.
- In refactors, prefer deleting code, merging duplicate paths, and simplifying
  control flow over introducing new abstractions. Avoid replacing straightforward
  code with a larger structure unless the tradeoff is explicit and worthwhile.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genome-spy/genome-spy](https://github.com/genome-spy/genome-spy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

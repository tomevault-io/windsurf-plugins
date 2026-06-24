---
trigger: always_on
description: JSX → Marked Template + client JS compiler. Signal-based reactivity for any backend.
---

# BarefootJS

JSX → Marked Template + client JS compiler. Signal-based reactivity for any backend.

## Project Setup / Tech Stack

This project primarily uses TypeScript with Go template adapters. Use `bun` instead of `npm` for package management. For CSS, use UnoCSS — note that UnoCSS alpha modifiers do not work with CSS variables, and files must be explicitly included in UnoCSS scanning config.

## Architecture

2-phase compilation: JSX → IR → Marked Template + Client JS.

- `packages/jsx/src/` — Core compiler. Key files: `jsx-to-ir.ts` (Phase 1), `ir-to-client-js.ts` (Phase 2), `analyzer.ts` (reactivity analysis).
- `packages/client/src/` — Client runtime (`createSignal`, `createEffect`, etc.) with DOM runtime under `./runtime`.
- Adapters: `packages/adapter-hono/` (Hono/JSX), `packages/adapter-go-template/` (Go `html/template`).

See `spec/compiler.md` for the full pipeline architecture, IR schema, transformation rules, adapter interface, and error codes.

## Code Conventions

- **Never parse imports (or any JS/TS syntax) with regex or string matching.** Regexes false-match inside string/template literals and comments, and miss multi-line clauses, trailing commas, and `import type`. Use the established structural patterns instead:
  - **Source files (.tsx/.ts)**: the IR's parsed metadata (`ir.metadata.imports`, built by the analyzer's TS AST walk — see `collectImport` in `packages/jsx/src/analyzer.ts`).
  - **Compiled client JS**: a TS AST walk over top-level statements (`ts.isImportDeclaration` + span-based splicing). Precedents: `packages/cli/src/lib/resolve-imports.ts` (migrated from regex to AST for exactly this reason — see `shapeFromDecl`) and `packages/jsx/src/combine-client-js.ts`.
  - Do not add a second parsing library (e.g. es-module-lexer) — `typescript` is already a direct dependency and the AST walk is the repo-wide idiom.
- **Never add compiler options/hooks for tool-specific output rewriting** (e.g. a rewrite callback on `CompileOptions`). Once such a hook exists it reads as a sanctioned extension point and accretes callers. Tools that need to adjust emitted client JS post-process it themselves with the TS AST walk above; an extra `ts.createSourceFile` parse is acceptable off the build hot path (e.g. `bf debug profile`), not in `bf build`.

## Testing

See `spec/testing.md` for full testing specification with APIs, patterns, and examples.

| Layer | Verifies | Location | Speed |
|-------|----------|----------|-------|
| Compiler unit | Transformation rules, error codes, analysis | `packages/jsx/src/__tests__/` | ms |
| Component IR | Structure, a11y, signals, classes, event wiring | `ui/components/ui/*/index.test.tsx` | ms |
| Adapter conformance | IR → HTML output per adapter | `packages/adapter-tests/fixtures/` | ms |
| CSR conformance | Client JS → correct DOM output | `packages/adapter-tests/src/__tests__/csr-conformance.test.ts` | ms |
| Runtime unit | Signals, DOM ops, hydration primitives | `packages/client/__tests__/` | ms |
| E2E | User interactions, hydration, visual | `site/ui/e2e/` | seconds |

Quick decision guide:
- **New UI component** → Component IR test using `renderToTest()`
- **Compiler internals** (analysis, error codes, codegen) → Compiler unit test
- **Template HTML output** → Adapter conformance fixture
- **Client JS behavior** → CSR conformance fixture
- **Click/keyboard behavior** → E2E test
- **Which handler calls which setter** (event→setter wiring) → Component IR test via `renderToTest().find(...).onClick`. This verifies the compiler-built dependency *path*, not the runtime value — assert the path here, assert the displayed value in E2E.
- **Static attribute / class / ARIA changes** → Component IR test. Do NOT add an E2E test for static-only changes; that's an anti-pattern (see `spec/testing.md`).
- **Hydration correctness** is a compiler invariant. Fix in `packages/jsx/`, verify with E2E.

`renderToTest` resolution limits (known): the IR analyzer does NOT resolve `Record<T, string>[key]` indexed lookups or default-prop values. For variant components (`const sizeClasses: Record<Size, string> = {...}` + `${sizeClasses[size]}`), the `.classes` array in IR only contains the base class tokens, not the per-variant ones. Verify variant resolution at the adapter conformance layer instead, or add a fixture in `packages/adapter-tests/fixtures/`. See `ui/components/ui/button/index.test.tsx` for the existing workaround pattern.

Tracked limitations across the compiler, adapters, and runtime live under the [`known-limitation`](https://github.com/piconic-ai/barefootjs/labels/known-limitation) label — that label URL is the source of truth. Adapter-internal declarations (`skipJsx`, `skipFixtures`, `expectedDiagnostics`) carry a docstring pointer back to the per-issue URL.

Workflow for editing a UI component:
1. Run `bun run bf docs <component>` (and `bf debug graph <component>` if `"use client"`) for the API surface.
2. Add or update the IR test (red).
3. Edit the component.
4. Re-run the IR test (green).
5. Update `site/ui/e2e/<component>.spec.ts` **only if** user-facing interactive behavior (click / keyboard / hover / hydration) changed.

## CLI


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [piconic-ai/barefootjs](https://github.com/piconic-ai/barefootjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->

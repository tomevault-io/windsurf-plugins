---
trigger: always_on
description: Infrastructure-as-Code deployment tool for Roblox, written in TypeScript. Modern
---

# Bedrock Project Context

## What is Bedrock?

Infrastructure-as-Code deployment tool for Roblox, written in TypeScript. Modern
replacement for [Mantle](https://github.com/blake-mealey/mantle) (Rust-based
tool no longer maintained).

## Architecture

- **Language**: TypeScript (ES modules)
- **Runtime**: Bun
- **Auth**: Roblox Open Cloud APIs only (no ROBLOSECURITY)
- **Toolchain**: Vite+ (`vp pack` builds, `vp test` runs Vitest, `vp run`
  orchestrates tasks)
- **Lint**: eslint from monorepo root only (`pnpm lint`), no per-package lint
  scripts

## Architecture Quick Reference

**Pattern**: FCIS (Functional Core, Imperative Shell) + Ports

```text
┌─────────────────────────────────────────────────────┐
│                      Shell                          │
│  (I/O, CLI commands, orchestration)                 │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │                   Core                       │   │
│  │  (Pure functions, business logic, no I/O)   │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │  Port    │  │  Port    │  │  Port    │         │
│  │ (State)  │  │(OpenCloud)│ │ (Config) │         │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘         │
└───────┼─────────────┼─────────────┼───────────────┘
        │             │             │
   ┌────▼────┐  ┌─────▼────┐  ┌────▼─────┐
   │ Adapter │  │ Adapter  │  │ Adapter  │
   │ (Gist)  │  │ (HTTP)   │  │  (c12)   │
   └─────────┘  └──────────┘  └──────────┘
```

- **Core**: Pure functions, no side effects, easy to test
- **Shell**: Orchestrates I/O, calls core with data from adapters
- **Ports**: Interfaces defining what adapters must implement
- **Adapters**: Concrete implementations (Gist, Open Cloud HTTP, etc.)

## Testing Requirements (NON-NEGOTIABLE)

Every line of production code must be written in response to a failing test.

**RED → GREEN → REFACTOR cycle:**

1. **RED:** Write failing test for desired behavior
2. **GREEN:** Write minimum code to pass
3. **REFACTOR:** Clean up while tests stay green

**Commit cadence:** The pre-commit hook (`hk`) runs lint, typecheck, test, and
build, so a pure-RED commit is rejected before it can land. Work RED → GREEN in
the working tree, then commit RED + GREEN **together** as one commit per
behaviour slice. REFACTOR lands as a separate commit only when refactoring adds
value for that slice.

**Git history must show TDD compliance — one commit per behaviour slice.**

**Test levels:**

| Layer    | Test with         | Isolation                                                                |
| -------- | ----------------- | ------------------------------------------------------------------------ |
| Core     | Unit tests        | None needed                                                              |
| Shell    | Integration tests | Fake adapters                                                            |
| Adapters | Adapter tests     | Injected fake transport (e.g. `fakeFetch`, `@bedrock-rbx/ocale/testing`) |
| E2E      | Scenario tests    | Real APIs                                                                |

**Coverage**: 100% required (statements, branches, functions, lines)

**Naming**: `it("should <behavior>")` - enforced by ESLint

**Anti-patterns (will be rejected):**

- Writing implementation before tests
- Testing mock behavior instead of real behavior
- Mocking without understanding dependencies
- Suppressing surviving mutants with `Stryker disable` directives. Kill the
  mutation with a test or refactor the code so a test can observe it.

### Public API examples

Add a JSDoc `@example` block to any symbol exported from a package's
`src/index.ts` barrel **where the example adds value** — i.e. the usage is
non-trivial, has surprising edge cases, or the return shape isn't obvious from
the signature. Skip `@example` on pass-through re-exports and trivial getters.

Examples are dual-purpose:

- `pnpm gen:example-tests` compiles every `@example` code block into an
  `it(...)` test in `<source>.example.spec.ts`. The generator prepends
  `import { expect, it } from "vitest";` as a file header, so source blocks
  **omit** that import -- include only the imports for the symbols being
  demonstrated. Each block should include `expect(...)` assertions that prove
  the claim.
- The same blocks are rendered into the public docs site by TypeDoc;
  `typedoc-plugin-replace-text` strips `expect(...)` lines (and any stray vitest
  import, as a safety net) so readers see a clean usage sample.

Format:

````ts
/**
 * @example
 * ```ts
 * import { myFn } from "@bedrock-rbx/pkg";
 *
 * const result = myFn({ foo: 1 });
 * expect(result).toEqual({ ok: true });
 * ```
 */
````

One well-chosen `@example` is the target. Add a second only when a single block
genuinely cannot convey the behavior — e.g. a success case plus a qualitatively
different failure mode. More examples are not better; resist the urge to
enumerate permutations of the same call. If you find yourself reaching for a
third block, the symbol probably needs clearer types or a split, not more docs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christopher-buss/bedrock](https://github.com/christopher-buss/bedrock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->

---
trigger: always_on
description: > **Update after learnings or mistakes** -- when a correction, new convention, or hard-won lesson emerges during development, append it to the relevant section of this file immediately. AGENTS.md is the source of truth for project conventions and should grow as the project does.
---

# ox -- Agent Guidelines

> **Update after learnings or mistakes** -- when a correction, new convention, or hard-won lesson emerges during development, append it to the relevant section of this file immediately. AGENTS.md is the source of truth for project conventions and should grow as the project does.

## TypeScript Conventions

- **Exact optional properties** -- `exactOptionalPropertyTypes` is enabled in tsconfig. Optional properties must include `| undefined` in their type if they can be assigned `undefined` (e.g. `foo?: string | undefined`, not `foo?: string`).
- **No unchecked indexed reads** -- `noUncheckedIndexedAccess` is enabled. Narrow indexed reads before use, or make the invariant obvious with the smallest possible assertion.
- **`readonly` arrays** -- use `readonly T[]` for array types in type definitions.
- **Existing `readonly` properties are fine** -- ox has DOM-shaped WebAuthn types, type snapshots, and inference-heavy literals that intentionally preserve `readonly` properties. Do not churn them just to satisfy a style preference.
- **`type` over `interface` by default** -- use `type` for project-owned shapes. Ambient declarations and DOM-shaped compatibility types may use `interface`.
- **`.js` extensions** -- all relative source imports include `.js` for ESM compatibility.
- **Follow local import style** -- ox uses both namespace imports and named internal imports. Match the surrounding file instead of mass-converting import lists.
- **Zod import aliases** -- import `zod/mini` as `z`, and import zod module namespaces as `z_<Module>` (for example, `import * as z_Hex from '../Hex.js'`).
- **Classes for errors only** -- all other APIs use functions and plain data.
- **Errors live next to the code that throws them** -- module-specific failure classes live inside the module that owns the failure mode. Place each error class near the bottom of the module so the public functions and types are what the reader sees first. Set `name` to the namespaced form (`'Hex.InvalidHexValueError'`, `'SignatureEnvelope.VerificationError'`, etc.).
- **No enums** -- use union types or `as const` objects for fixed sets.
- **camelCase constants** -- prefer `camelCase` for local constants unless the surrounding file already uses protocol-style uppercase names for numeric constants.
- **`const` generic modifier** -- use to preserve literal types for full inference.
- **Options default `= {}`** -- use `options: Options = {}` not `options?: Options`.
- **Namespace params and return types** -- place function parameter, return, and error types in a `declare namespace` matching the function name (e.g. `from.Options`, `serialize.ErrorType`). Do not lift the params type to a sibling export unless the surrounding module already has a shared type.
- **`options` over `args`** -- use `options` for typed option bags. Use domain nouns only when the parameter is not an options bag.
- **Minimal variable names** -- prefer short, obvious names. Use `options` not `serializeOptions`, `fn` not `callbackFunction`, etc. Context makes meaning clear.
- **No redundant type annotations** -- if the return type of a function already covers it, do not annotate intermediate variables. Let the return type do the work.
- **No inline object types on locals** -- when a local variable needs an explicit object-type annotation, declare a named `type` on the line directly above and reference it.
- **Return directly** -- do not declare a variable just to return it. Use `return { ... }` unless the variable is needed for reuse or readability.
- **IIFE expressions for fallible local derivations** -- when a local needs `try`/`catch` to parse or normalize a value, prefer an IIFE expression over `let value: T` followed by assignment inside `try`.
- **Skip braces for single-statement blocks** -- omit `{}` for single-statement `if`, `for`, etc., when the surrounding file follows that style.
- **No section separator comments** -- do not use `// ---` or `// ===` divider comments. Let JSDoc and whitespace provide structure.
- **Static imports by default** -- use static `import` declarations. Dynamic imports are reserved for real runtime boundaries already present in ox, such as worker or WASM loading.
- **Minimize `as any`** -- avoid new `as any` where a safer assertion is practical, but do not mass-rewrite existing crypto, tuple, and inference glue that already relies on it.
- **Destructure when accessing multiple properties** -- prefer `const { a, b } = options` over repeated `options.a`, `options.b`.
- **Read from `options.x` when normalizing a single field** -- when transforming exactly one option into a local of the same name, read it directly from `options` instead of destructuring and inventing a second name.
- **Hex helpers over ad hoc conversion** -- use ox helpers like `Hex.fromNumber`, `Hex.toBytes`, `Bytes.fromHex`, `Value.fromGwei`, etc. instead of open-coded conversions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wevm/ox](https://github.com/wevm/ox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

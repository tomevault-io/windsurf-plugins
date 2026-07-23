---
trigger: always_on
description: - Prefer pure functions that return values over void functions with side effects.
---

# AGENTS.md

## Code Style Preferences

- Prefer pure functions that return values over void functions with side effects.
- Functions should return results rather than mutating state when possible.

## Code Comment Policy

- Do NOT add comments unless they are critical and required.
- Only add comments when they document:
  - Workarounds for bugs or limitations
  - Dangerous side effects
  - Non-obvious behavior that could cause issues

## Optimization Guidelines

- DO NOT extract helper functions unless they provide SIGNIFICANT net line reduction (at least 20+ lines saved).
- DO NOT refactor code just to "reduce duplication" if the net change is negligible (e.g., -2 lines).
- Extracting helpers that save only a few lines is NOT an improvement - it just moves code around.
- The original explicit code is often more readable than abstracted helpers.
- Focus on changes that have REAL impact: performance improvements, actual deletions, fixing bugs.

## Code Placement & Refactoring

Where a declaration lives is part of the design. These rules decide it; they are behavior-neutral —
a move builds and passes `swift test` (SwiftSync) **and** `swift test` (DemoCore) both before and after.

### File naming

- A file is named after a type that **actually exists in it** (`SyncPayload.swift` holds `SyncPayload`) —
  never a concept or a phantom type (a `…Store.swift`/`…Manager.swift` with no such type in it). Don't
  let a file become a grab-bag of unrelated types — split such a file one public type per file. There is
  no `Core`/`Misc`/`Helpers`-style catch-all.
- `Type+Feature.swift` is **only** for extending a type you don't own (a stdlib/framework type from
  another module): `String+SnakeCase.swift`, `DateFormatter+Sync.swift`, `ModelContext+Sync.swift`.
- An extension on a type defined in **this** module is part of that type's definition, **not** a
  `+Feature` file: a protocol's default-impl extension, a type's `LocalizedError` conformance, and a
  small helper extension all live in the *same file as the type* they extend. Never hoist them out.

### No free functions — every function has an owner

- Home an internal helper on the type it naturally operates on, as an extension (snake-casing →
  `String`; a related-row fetch → `ModelContext`; dedupe → `Array`/`Sequence`).
- When the function has **no single operand** (it takes a payload *and* a model, or is an
  identity/namespace-level operation), home it as a `static` on the `SwiftSync` namespace enum.
- **Public macro-SPI** (functions `@Syncable`-generated code calls cross-module — must be `public`)
  goes on `SwiftSync` statics, **never** on a stdlib/model type. Homing it on `Dictionary`/the model
  protocol would force *that type's* public API to carry sync internals for every consumer.
- The only allowed free function is a control-flow wrapper with genuinely no operand that reads worse
  namespaced, kept `internal` so it pollutes nothing (`syncPerformanceProfile`). Justify it explicitly or don't.

### When a type earns its own file vs. folds into a caller

Measure first — `grep -rn TypeName` repo-wide — and **distinguish library `Sources` references (which
decide the home) from test/consumer references (which are usage, not a home)**. Verify the claim; don't
assume from a type's name what calls it.

- **Multiple library callers** → its own file — *unless* the type is a subsystem's vocabulary or a
  protocol's parameter (a profiler's phase enum, an `OptionSet` a protocol method takes); that lives in
  the owner's file however many sites reference it. Call-site count guards against cramming into an
  *arbitrary* caller, not against co-locating with the conceptual owner. And if folding would bury a
  cohesive subsystem in a namespace/catch-all file, do the **reverse** — move the small installer/glue
  into the subsystem's own well-named file.
- **Exactly one library caller** → fold it into that caller's file (`SyncPayloadConvertible` → only the
  `SyncContainer.sync` overloads reference it; `SyncRelationshipSchemaDescriptor` → only the
  `SyncModelable` requirement). A public type still conformed to by
  consumers is fine to relocate — moving the *declaration* next to its one library caller doesn't change
  the public surface.
- **Zero library callers** (public API exercised only by consumers or by `@Syncable`-generated code) →
  categorize by *what calls it*, not by a runtime caller:
  - macro-generated-code SPI (e.g. `ExportState`, called only by the generated `export()`) homes with
    its SPI siblings in `MacroRuntimeSupport.swift` — same category as `exportEncodeValue`/`exportSetValue`.
  - a model-family protocol consumers conform to by hand keeps its own file alongside `SyncModelable`/
    `SyncUpdatableModel` — not macro-generated, so the macro file is the wrong home. But first confirm it
    is genuinely consumer-facing: a public protocol with zero library callers that the library never
    dispatches on is dead surface to remove, not a seam to keep.

- **Duplicated parallel logic is a correctness hazard, not just clutter.** When two types carry the same
  logic (a SwiftUI observer mirroring a plain publisher; a stub/real overload pair), a fix that lands in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [3lvis/SwiftSync](https://github.com/3lvis/SwiftSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

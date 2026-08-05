---
trigger: always_on
description: Guidance for AI agents working in the `substrait-java` repository. This complements
---

# AGENTS.md

Guidance for AI agents working in the `substrait-java` repository. This complements
`CONTRIBUTING.md` (commit conventions, style guide) with practical, codebase-specific
knowledge.

## What this project is

`substrait-java` is the Java implementation of [Substrait](https://substrait.io/) —
a cross-language specification for relational query plans. It provides an immutable
POJO model for plans/relations/expressions/types and bidirectional conversion to and
from the Substrait protobuf wire format, plus integrations (Isthmus → Apache Calcite,
Spark).

## Module layout

| Module | Path | Purpose |
| --- | --- | --- |
| `:core` | `core/` | POJO model, proto converters, function extension handling. The heart of the project. |
| `:isthmus` | `isthmus/` | Calcite SQL ⇄ Substrait conversion. |
| `:spark` | `spark/` + variants `spark-3.4_2.12`, `spark-3.5_2.12`, `spark-4.0_2.13` | Spark plan ⇄ Substrait (Scala). Source shared in `spark/src`; each variant compiles it against a different Spark/Scala version — see Spark notes. |
| `:examples` | `examples/isthmus-api`, `examples/substrait-spark` | Runnable examples. |
| `:isthmus-cli` | `isthmus-cli/` | CLI over `:isthmus`, compiled to a GraalVM **native image** (`nativeCompile`) — the `isthmus` binary + smoke tests. |
| `build-logic` | `build-logic/` | Gradle **included build**: Kotlin-DSL convention plugins (`substrait.java-conventions` → shared Java config + PMD). Has its own `gradle.properties`; does not inherit the root's. |

The `substrait/` directory is a **git submodule** of the upstream spec, and several
`:core` build inputs are read from it — none are vendored in this repo, so don't look
for them under `core/src`:

- **Proto**: `substrait/proto/substrait/*.proto` → generated Java in package `io.substrait.proto`.
- **ANTLR grammars**: `substrait/grammar/*.g4` (notably `SubstraitType.g4`, the type /
  function-signature grammar) → parser generated under `io.substrait.type`.
- **Standard extension YAMLs**: `substrait/extensions/*.yaml` (e.g. `functions_arithmetic.yaml`)
  → packaged into `:core` resources at `substrait/extensions/`.
- **Validation schemas**: `substrait/text/simple_extensions_schema.yaml` and
  `dialect_schema.yaml` → validate extensions/dialects (mainly in tests).

`:core:submodulesUpdate` (submodule init/update) therefore gates proto and grammar
generation. These files are owned by the upstream spec — to change them, update
`substrait` upstream and bump the submodule pin, don't edit them in-tree.

## Core architecture (the pattern most changes follow)

The POJO model uses **Immutables** (`org.immutables:value`):

- Interfaces/abstract classes are annotated `@Value.Immutable`; the enclosing type
  (e.g. `Expression`) is `@Value.Enclosing`. The build generates
  `ImmutableExpression.Foo` etc. **These generated classes do not exist until you
  compile**, so your IDE/diagnostics will show "cannot be resolved" errors for
  `ImmutableExpression.*` and new `builder()` methods until `:core:compileJava` runs.
  This is expected — compile to confirm.
- Each POJO exposes a static `builder()` delegating to the generated immutable.
- Immutables copies an accessor's Javadoc **verbatim** into the generated
  `ImmutableXxx` (which lives in the same package as the abstract type). A
  `{@link}`/`@see` targeting a type in a *different* package resolves in the source
  file (which imports it) but NOT in the generated file, so `:core:javadoc` fails with
  `reference not found`. **Fully-qualify** cross-package `{@link}` targets in the
  Javadoc of any `@Value.Immutable` accessor.

Expressions are visited via a double-dispatch **visitor**:

- `ExpressionVisitor` (interface) — one `visit(...)` overload per concrete expression
  type. Direct implementors **must** implement every method.
- `AbstractExpressionVisitor` — provides `visitFallback`-based defaults for every
  method, so subclasses only override what they need. Implementors that extend this
  (e.g. isthmus `ExpressionRexConverter`, spark `DefaultExpressionVisitor`) do **not**
  break when a new expression type is added.
- Adding a new expression type therefore means: add the POJO, add a `visit` method to
  `ExpressionVisitor`, add a default to `AbstractExpressionVisitor`, and update the
  **direct** implementors (`ExpressionProtoConverter`, `ExpressionCopyOnWriteVisitor`,
  examples `ExpressionStringify`).

The same double-dispatch pattern recurs across the model, so the "add a case → update the
direct implementors" rule applies well beyond expressions:

- **Relations**: `RelVisitor` / `AbstractRelVisitor` — same `*Visitor` + `Abstract*Visitor`
  shape as expressions — plus copy-on-write transformers `RelCopyOnWriteVisitor` and
  `ExpressionCopyOnWriteVisitor`.
- **Types**: `TypeVisitor`, extended by `ParameterizedTypeVisitor` / `TypeExpressionVisitor`
  for function-signature and derived-type expressions. Interface-only (no `Abstract*Visitor`
  fallback), so implementors must handle every type kind.
- **Function arguments**: a `FunctionArg` is an `Expression`, `Type`, or `EnumArg`, dispatched
  via `FunctionArg.accept(fnDef, argIdx, FuncArgVisitor, ctx)`; the nested `FuncArgVisitor`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [substrait-io/substrait-java](https://github.com/substrait-io/substrait-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->

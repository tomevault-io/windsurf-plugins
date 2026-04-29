---
trigger: always_on
description: **Kindlings** provides type classes (re)implemented with [Hearth](https://github.com/kubuszok/hearth/) to kick-off the ecosystem — cross-compiled (Scala 2.13 + 3) type class derivation using Hearth's macro-agnostic APIs.
---

# AGENTS

## Project Overview

**Kindlings** provides type classes (re)implemented with [Hearth](https://github.com/kubuszok/hearth/) to kick-off the ecosystem — cross-compiled (Scala 2.13 + 3) type class derivation using Hearth's macro-agnostic APIs.

- **Scala Versions**: 2.13.18, 3.8.2 | **Platforms**: JVM, Scala.js, Scala Native | **Build Tool**: SBT

## MCP — Always verify APIs via MCP before using them

Metals MCP at `.metals/mcp.json` (`kindlings-metals`). Query definitions/types/symbols, check compilation errors before running sbt. Fix MCP issues before running sbt to avoid long feedback loops.

## Global Rules

 - **Always use `sbt --client`** — never bare `sbt`. Connects to running server instead of launching new JVM.
 - **Redirect sbt output**: `sbt --client "module/clean ; test-jvm-2_13" 2>&1 | tee /tmp/sbt-output.txt`
   Then inspect with `grep`/`tail`/`head`. Only re-run sbt if code was modified.
 - Only run `sbt compile`/`sbt test` after MCP shows no compilation issues.
 - **Do not modify `dev.properties`** — used by the developer for IDE platform focus.
 - **Do not add yourself as co-author** — no `Co-Authored-By: Claude ...` in commits.

### Project matrix structure

Uses **sbt-projectmatrix** (not sbt-crossproject). **Do NOT use** `++` version switching or `scalaCrossVersion`.

**Naming**: no suffix = Scala 2.13 JVM, `3` = Scala 3 JVM, `JS`/`JS3` = Scala.js, `Native`/`Native3` = Scala Native.
Example: `fastShowPretty` (2.13 JVM), `fastShowPretty3` (3 JVM), `fastShowPrettyJS3` (3 JS).

### Test aliases

| `test-jvm-2_13` | `test-jvm-3` | `test-js-2_13` | `test-js-3` | `test-native-2_13` | `test-native-3` |
|---|---|---|---|---|---|

**JVM-only by default** — JS/Native are slow (Native can OOM). CI covers all platforms.

## Incremental Compilation Gotchas

 - **Always clean after macro changes** — incremental compilation does NOT re-expand macros.
 - Clean specific module: `sbt --client "module/clean ; module3/clean ; test-jvm-2_13 ; test-jvm-3"`
 - Nuclear option: `sbt --client clean` then `sbt --client "test-jvm-2_13 ; test-jvm-3"`

## Cross-Quotes and Macro-Agnostic APIs

Code uses `Expr`, `Type`, etc. from **Hearth's API**, NOT `scala.quoted.Expr` or `c.Expr`:
- `Expr.quote { ... Expr.splice { ... } }` — cross-platform quotes/splices
- `Type.of[A]`, `Type.Ctor1.of[F]` — cross-platform type representations
- Traits mix in `MacroCommons` and `StdExtensions` for the full API
- Scala 3: `inline def` + `${ ... }` with `MacroCommonsScala3`; Scala 2: `def ... = macro` with `MacroCommonsScala2`

## Cross-Compilation Pitfalls

See `docs/contributing/cross-compilation-pitfalls-skill.md` for the full live list.
The most load-bearing entries to keep in mind for any macro work:

- **Sibling `Expr.splice` isolation (Scala 3)** — each splice gets its own `Quotes`; for
  multi-method type-class instances **derive everything at `Q0` using `ValDefsCache`,
  then call cached `def`s by name from inside each splice**. See
  `docs/contributing/def-caching-skill.md`. Do **not** use `LambdaBuilder` as a
  workaround — that was a misdiagnosis; the real fix is `cache.toValDefs.use` placement.
- **`summonExprIgnoring` vs OOM** — always pass the library auto-derivation methods to
  ignore, otherwise summoning recurses into them and OOMs the compiler.
- **`IsMap` before `IsCollection`** — `Map <: Iterable`, so check `IsMap` first.
- **`Type.Ctor2.of[Function1].unapply` wrong on Scala 3** — wrap with
  `Type.Ctor2.fromUntyped[Function1](impl.asUntyped)`.
- **`Type.of[A]` bootstrap cycle in extensions** — see the pitfalls skill.

## Standard extensions, `LambdaBuilder`, and def-caching rules

- **Load standard extensions exactly once per macro bundle.** See
  `docs/contributing/load-standard-extensions-skill.md`. The shared implementation
  lives in `derivation-commons` (`hearth.kindlings.derivation.compiletime.LoadStandardExtensionsOnce`);
  per-module thin delegates re-export it in each module's `internal.compiletime` package.
  Use `ensureStandardExtensionsLoaded()` instead of calling
  `Environment.loadStandardExtensions()` directly. Issue `kubuszok/kindlings#65`.
- **`LambdaBuilder` only for collection/Optional iteration lambdas.** Strict rule, no
  exceptions. See `docs/contributing/lambda-builder-when-to-use-skill.md`. For
  multi-method type-class instances or any other shape that needs pre-derived helper
  functions, use the def-caching pattern instead.
- **Def-caching is the answer to multi-method instances and sibling-splice isolation.**
  See `docs/contributing/def-caching-skill.md`. The recipe: shared `ValDefsCache`,
  derivation in `Q0` via `forwardDeclare` + `buildCachedWith`, helper-call functions
  retrieved via `cache.getNAry`, outer `cacheState.toValDefs.use` wrapping the entire
  `Expr.quote { new T[A] { … } }`. Reference implementations: `cats-derivation`
  `HashMacrosImpl` (two-method) and `jsoniter-derivation`
  `deriveCombinedCodecTypeClass` (five-method, parallel derivation via `parTuple`).
- **Multi-method derivation must use `parTuple` for error aggregation.** Sequential

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubuszok/kindlings](https://github.com/kubuszok/kindlings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

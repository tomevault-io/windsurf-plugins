---
trigger: always_on
description: This repository packages the MuleSoft DataWeave runtime as GraalVM native artifacts.
---

# AGENTS.md

## Purpose

This repository packages the MuleSoft DataWeave runtime as GraalVM native artifacts.
The language/runtime itself is supplied by pinned `org.mule.weave:*` dependencies;
language implementation changes generally belong in the upstream DataWeave repository.

- `native-cli`: native `dw` executable. Java/picocli parses arguments; Scala implements commands.
- `native-lib`: native `dwlib` shared library, C ABI, Python binding, and Node binding.
- `native-cli-integration-tests`: downloaded suites executed against the compiled native CLI.
- `benchmarks/`: shared corpus/schema and CLI, JVM, Node, and Python benchmark runners.

Key entry points are `DWCLI.java`, `NativeRuntime.scala`, `NativeLib.java`, and
`ScriptRuntime.java`. Read the nearest module README before broad changes.

## Toolchain

- Use the checked-in `./gradlew` wrapper.
- Native builds require GraalVM Community Java 24 with `native-image`.
- Set `GRAALVM_HOME` and `JAVA_HOME`; native-image tasks need about 6 GB heap.
- Node bindings require Node.js 18+ and a C compiler/node-gyp toolchain.
- Python bindings support Python 3.9+.
- Java source/target compatibility is 17; Scala is 2.12.

## Build Commands

Run from the repository root unless a command says otherwise.

```bash
./gradlew native-cli:test
./gradlew native-lib:test -PskipNodeTests=true -PskipPythonTests=true
./gradlew native-cli:nativeCompile
./gradlew native-lib:nativeCompile
./gradlew build -PskipNodeTests=true
./gradlew native-cli:distro
./gradlew native-lib:buildPythonWheel
./gradlew native-lib:buildNodePackage
./gradlew clean
```

Native outputs are `native-cli/build/native/nativeCompile/dw` and
`native-lib/build/native/nativeCompile/dwlib.{dylib,so,dll}`. Useful flags:
`-PskipNodeTests=true`, `-PskipPythonTests=true`, `-PskipStripDebug=true`, and
`-PpythonExe=/path/to/python`.

## Test Commands

```bash
./gradlew native-cli:test
# One ScalaTest suite (preferred single-test granularity)
./gradlew native-cli:test --tests "org.mule.weave.dwnative.cli.DataWeaveCLITest"
# One native-lib JUnit class or method; skip unrelated binding lanes
./gradlew native-lib:test --tests "org.mule.weave.lib.ScriptRuntimeTest" \
  -PskipNodeTests=true -PskipPythonTests=true
./gradlew native-lib:test --tests "org.mule.weave.lib.ScriptRuntimeTest.runSimpleScript" \
  -PskipNodeTests=true -PskipPythonTests=true
# Native CLI integration/TCK suite (builds and runs the native binary)
./gradlew -PweaveTestSuiteVersion=2.10.0 -DweaveSuiteVersion=2.10.0 \
  native-cli-integration-tests:test
./gradlew native-cli-integration-tests:test \
  --tests "org.mule.weave.clinative.NativeCliTest"
# Dependency-free benchmark JavaScript tests
./gradlew native-lib:benchmarkJsUnitTest
cd benchmarks && node --test lib/stats.test.mjs
cd benchmarks && node --test --test-name-pattern="computeStats returns" lib/stats.test.mjs
```

Node/Vitest commands (run after native staging/build when using integration tests):

```bash
cd native-lib/node
npm install
npm run build
npm test
npm run test:unit
npm run test:integration
npm test -- tests/unit/result.test.ts
npm test -- tests/integration/dataweave.test.ts -t "basic arithmetic"
```

Python binding tests are a custom executable script, not a configured pytest suite:

```bash
./gradlew native-lib:pythonTest
cd native-lib/python && python3 tests/test_dataweave_module.py
cd benchmarks/runners/python
python3 -m unittest test_bench.TestStats.test_matches_lib_stats_on_1_to_100
```

Node TCK is separate from normal `nodeTest`: run `./gradlew native-lib:stageTckSuites`,
then `cd native-lib/node && npm run test:tck`.

## Lint and Formatting

There is no repository-wide lint or formatter command: no Scalafmt, Checkstyle,
Spotless, ESLint, Prettier, Black, or Ruff configuration is checked in. Do not claim a
lint step exists or introduce formatting churn. Match neighboring code. For TypeScript,
`cd native-lib/node && npm run build:ts` is the configured strict type check.

No `.cursorrules`, `.cursor/rules/`, or `.github/copilot-instructions.md` rules exist.

## Code Style

- Preserve local import grouping; use explicit imports and avoid new wildcards.
- Java: four spaces, braces on the declaration line, braced control flow, explicit
  types, `PascalCase` classes, `camelCase` members, `UPPER_SNAKE_CASE` constants.
- Scala: two spaces, prefer `val`, idiomatic `foreach { x => ... }`, explicit public
  return types, case classes/sealed traits for domain variants, no semicolons.
- TypeScript/ESM: two spaces, double quotes, semicolons, trailing commas in multiline
  constructs, `camelCase` values, `PascalCase` types/classes.
- TypeScript is strict and emits CommonJS. Use `node:` built-in imports and `import type`;
  local `.ts` imports omit extensions.
- Benchmark `.mjs` is ESM: include `.mjs` extensions and use `import.meta.url`, never
  `require` or CommonJS-only assumptions.
- Python: PEP 8 naming, four spaces, type hints on public/boundary APIs, dataclasses for
  result/input models, and snake_case public names translated to camelCase wire fields.
- Gradle formatting varies by file; preserve the file's indentation and quote style.
- Add comments/Javadoc/TSDoc for public APIs, ABI contracts, ownership, concurrency, or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mulesoft/data-weave-cli](https://github.com/mulesoft/data-weave-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->

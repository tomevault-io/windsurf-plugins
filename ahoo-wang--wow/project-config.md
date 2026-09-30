---
trigger: always_on
description: <!-- This file provides coding agents with context about this package. -->
---

# AGENTS.md — @ahoo-wang/wow-generator

<!-- This file provides coding agents with context about this package. -->

## Build & Run Commands

```bash
# Build this package
pnpm --filter @ahoo-wang/wow-generator build

# Run tests
pnpm --filter @ahoo-wang/wow-generator test

# Type-check src and every test file (`test` runs it after vitest)
pnpm --filter @ahoo-wang/wow-generator test:type

# Run a single test file
pnpm --filter @ahoo-wang/wow-generator exec vitest run --maxWorkers=2 test/pipeline/codeGenerator.test.ts

# Accept a change to the public surface
pnpm --filter @ahoo-wang/wow-generator exec vitest run test/publicSurface.test.ts -u

# Lint
pnpm --filter @ahoo-wang/wow-generator lint

# Clean
pnpm --filter @ahoo-wang/wow-generator clean

# Run CLI (after build)
node dist/cli.js generate -i <openapi-spec> -o <output-dir> -t tsconfig.json
```

## Testing

- Vitest with `globals: true` and `@vitest/coverage-v8`; `--testTimeout=15000`. Always pass `--maxWorkers=2`
  locally. Coverage thresholds (statements / branches / functions / lines): 97 / 92.5 / 98.5 / 98.
- Tests live in `test/`, grouped by the behaviour they hold, not by the class that implements it:

| Directory                     | Holds                                                                                                   |
| ----------------------------- | ------------------------------------------------------------------------------------------------------- |
| `test/goldens/`               | Generated output byte for byte: the Wow documents, the OpenAI document, determinism across locales      |
| `test/probes/`                | A small document through the CLI into a cold directory, then `tsc --strict` or a real run               |
| `test/models/`                | What a generated model's types admit, checked by type-checking assignments against them                 |
| `test/analysis/`              | The `GenerationModel` a document yields: names, files, parameter order, body and return kinds, warnings |
| `test/emitters/`              | The code each emitter writes for a document                                                             |
| `test/output/`                | `OutputStore`, and regenerating over earlier output on disk                                             |
| `test/cli/`, `test/pipeline/` | The CLI (options, exit codes, Ctrl-C) and `CodeGenerator` end to end                                    |
| the rest                      | The leaf modules they are named after (`openapi/`, `naming/`, `input/`, `types/`, `wow/`, `emit/`, …)   |

- `test/support/` builds documents (`specs.ts`), runs the analysis and the emitters in memory
  (`emission.ts`), writes and type-checks models (`models.ts`), and generates through the CLI (`generation.ts`,
  `probes.ts`).
- Coverage instruments TypeScript's own compiler as it runs, about five times slower. A new ts-morph project
  or `ts.createProgram` parses `lib.dom.d.ts` and the `node_modules` declarations first, so a test that builds
  one per check spends nearly all its time parsing: `models.ts` keeps one in-memory project per test file,
  `generateCold` generates into one shared project, and `typeCheck` caches library declarations. Keep new
  tests on these helpers, and keep a file at a few seconds so the files spread over the workers.

## Public surface

`src/index.ts` is the only code entry, and it only re-exports: the programmatic API (`CodeGenerator`, its
options and result, the loggers, `GeneratorError` and `EXIT_CODES`). The public types live in `src/api/`,
which imports nothing else from the package; `CodeGenerator` lives in `src/pipeline/codeGenerator.ts`. Its
exports are listed name by name in `test/surface/root.txt`, which `test/publicSurface.test.ts` writes from
the source; a new export, or a removed one, changes the list, and the change is made on purpose with `-u`
and called out in the PR. The build runs `scripts/verify-package.mjs`, which holds the built ES module and
CommonJS entries to the same list, checks the `bin` files exist, that the declarations reachable from the
entry import neither `ts-morph` nor `@ahoo-wang/fetcher-openapi`, that no built file loads an `@ahoo-wang`
package, that no declaration map ships, that no file under `dist` holds `devDependencies`, `catalog:` or
`workspace:`, and that the CLI of the packed tarball, unpacked in a temporary directory, prints package.json's
version for `--version`. The CLI takes that version from `src/version.ts`, which the build's `define` fills from
package.json; never import package.json from the source, which would bundle the whole manifest. Anything else under `src/` is internal: do not export it from
`src/index.ts`.

- `CodeGenerator` takes only its options. The package hands it more under symbols it does not export, the
  `Seams` of `src/pipeline/seams.ts`: `PROJECT_SEAM`, a ts-morph project to write into (tests use
  `createCodeGenerator(options, project)` from `test/support/generation.ts`), and `SIGNAL_SEAM`, the
  `AbortSignal` the CLI aborts on Ctrl-C.
- A `Logger` has four required methods, `debug`, `info`, `warn` and `error`. Details go to `debug`, the
  outcome of a run to `info`. Only the pipeline logs warnings: every stage returns its warnings as lines,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ahoo-Wang/Wow](https://github.com/Ahoo-Wang/Wow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->

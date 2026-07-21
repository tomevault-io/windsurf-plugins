---
trigger: always_on
description: Short reference for AI agents. For task-specific guidance (directives, integration tests), load skills from *
---

# AGENTS.md — Guidance for AI agents contributing to Scala CLI

Short reference for AI agents. For task-specific guidance (directives, integration tests), load skills from *
*[agentskills/](agentskills/)** when relevant.

> **LLM Policy**: All AI-assisted contributions must comply with the
> [LLM usage policy](https://github.com/scala/scala3/blob/HEAD/LLM_POLICY.md). The contributor (human) is responsible
> for every line. State LLM usage in the PR description. See [LLM_POLICY.md](LLM_POLICY.md).

## Human-facing docs

- **[DEV.md](DEV.md)**                   — Setup, run from source, tests, launchers, GraalVM.
- **[CONTRIBUTING.md](CONTRIBUTING.md)** — PR workflow, formatting, reference doc generation.
- **[INTERNALS.md](INTERNALS.md)**       — Modules, `Inputs → Sources → Build`, preprocessing.

## Build system

The project uses [Mill](https://mill-build.org/). Mill launchers ship with the repo (`./mill`). JVM 17 required.
Cross-compilation: default `Scala.defaultInternal`; `[]` = default version, `[_]` = all.

### Key build files

| File                            | Purpose                                                                                  |
|---------------------------------|------------------------------------------------------------------------------------------|
| `build.mill`                    | Root build definition: all module declarations, CI helper tasks, integration test wiring |
| `project/deps/package.mill`     | Dependency versions and definitions (`Deps`, `Scala`, `Java` objects)                    |
| `project/settings/package.mill` | Shared traits, utils (`HasTests`, `CliLaunchers`, `FormatNativeImageConf`, etc.)         |
| `project/publish/package.mill`  | Publishing settings                                                                      |
| `project/website/package.mill`  | Website-related build tasks                                                              |

### Essential commands

```bash
./mill -i clean                                                           # Clean Mill context
./mill -i scala …args…                                                    # Run Scala CLI from source
./mill -i __.compile                                                      # Compile everything
./mill -i unitTests                                                       # All unit tests
./mill -i 'build-module[].test'                                           # Unit tests for a specific module
./mill -i 'build-module[].test' 'scala.build.tests.BuildTestsScalac.*'    # Filter by suite
./mill -i 'build-module[].test' 'scala.build.tests.BuildTests.simple'     # Single test by name
./mill -i integration.test.jvm                                            # Integration tests (JVM launcher)
./mill -i integration.test.jvm 'scala.cli.integration.RunTestsDefault.*'  # Integration: filter by suite
./mill -i 'generate-reference-doc[]'.run                                  # Regenerate reference docs
./mill -i __.fix                                                          # Fix import ordering (scalafix)
scala-cli fmt .                                                           # Format all code (scalafmt)
```

## Project modules

Modules live under `modules/`. The dependency graph flows roughly as:

```
specification-level → config → core → options → directives → build-module → cli
                                                    ↑
                                          directives-parser
```

### Module overview

The list below may not be exhaustive — check `modules/` and `build.mill` for the current set.

| Module                                        | Purpose                                                                                                          |
|-----------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `specification-level`                         | Defines `SpecificationLevel` (MUST / SHOULD / IMPLEMENTATION / RESTRICTED / EXPERIMENTAL) for SIP-46 compliance. |
| `config`                                      | Scala CLI configuration keys and persistence.                                                                    |
| `build-macros`                                | Compile-time macros (e.g. `EitherCps`).                                                                          |
| `core`                                        | Core types: `Inputs`, `Sources`, build constants, Bloop integration, JVM/JS/Native tooling.                      |
| `options`                                     | `BuildOptions`, `SharedOptions`, and all option types.                                                           |
| `directives-parser`                           | Pure Scala 3 parser for `//> using` directive syntax: comment extraction, lexing, and parsing into AST nodes.    |
| `directives`                                  | Using directive handlers — the bridge between `//> using` directives and `BuildOptions`.                         |
| `build-module` (aliased from `build` in mill) | The main build pipeline: preprocessing, compilation, post-processing. Most business logic lives here.            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VirtusLab/scala-cli](https://github.com/VirtusLab/scala-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

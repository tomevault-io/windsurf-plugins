---
trigger: always_on
description: Tenzir is a data pipeline engine for security teams. It collects, parses,
---

# Tenzir

Tenzir is a data pipeline engine for security teams. It collects, parses,
shapes, enriches, and routes security telemetry with a unified dataflow
language. The Tenzir Query Language (TQL) is a domain-specific language for
building modular pipelines that process structured event data.

### Project structure

- `.agents/` - Agent-facing reference material
- `.docs/` - Optional local clone of the `tenzir/content` repository
- `.github/` - GitHub configuration and CI/CD workflows
- `changelog/` - Changelog entries and release metadata
  - Managed with the `tenzir-ship` framework
  - Reference: https://tenzir.com/docs/reference/ship-framework.md
- `cmake/` - CMake build system modules and utilities
  - Various CMake modules for dependencies
  - TenzirConfig.cmake.in - CMake configuration template
  - TenzirRegisterPlugin.cmake - Plugin registration utilities
- `libtenzir/` - Core Tenzir library
  - `include/tenzir/` - Public headers
  - `src/` - Implementation files
  - `builtins/` - Built-in operators, functions, formats, connectors, and related plugins
  - `aux/` - Vendored dependencies (CAF, simdjson, etc.)
  - `fbs/` - FlatBuffer schema definitions
  - `test/` - Unit tests
- `libtenzir_test/` - Test utilities library
- `nix/` - Nix package management
  - `package.nix` - Main package definition
  - `overlay.nix` - Nix overlay for custom packages
- `plugins/` - Additional plugins outside `libtenzir/builtins`
  - Each subdirectory is a plugin (amqp, kafka, s3, etc.)
- `python/` - Python packages and tooling
- `scripts/` - Utility and maintenance scripts
  - Platform-specific installation scripts
  - Development tools and helpers
  - Analysis and debugging utilities
- `tenzir/` - Main executable
  - `tenzir.cpp` - Main entry point
  - `services/` - System service configurations
- `test/` - Integration tests and expected outputs

## Key tasks

### Set up the checkout

Use checked-in entry points instead of invoking formatter binaries from `PATH`.
Tool versions are pinned in `lefthook.yml` or by the Nix formatter environment.

For non-Nix setups, install local hooks with:

```sh
npx --yes lefthook install
```

For Nix setups, install local hooks from the dev shell with:

```sh
lefthook install
```

After installing hooks, Git runs Lefthook's `pre-push` hook automatically. To
auto-fix local formatting issues, run Lefthook's `fix` hook on the files changed
by your branch. Pass files explicitly with `--file <path>` when running
Lefthook outside Git's actual pre-push flow.

Nix setups can also run the repository formatter with
`nix run .#format -- <path>...`. CI uses this Nix/treefmt path on PR-changed
files, so use it to reproduce and fix CI style failures.

### Configure the build

Configure a build by selecting a CMake prefix:

```sh
cmake --list-presets
cmake --preset <preset>
```

A build is configured when its directory contains `CMakeCache.txt`.

Before configuring a new build, run `scripts/build.sh --print-build-dir` to find
an existing configured build. Reuse it unless the task specifically requires a
different configuration.

### Compile the project

Compile a configured build:

```sh
scripts/build.sh
```

Pass a target only when needed:

```sh
scripts/build.sh tenzir-unit-test
```

### Run pipelines

Use the `tenzir` binary to execute a TQL program:

- The first argument is the pipeline definition.
- Alternatively, pass the pipeline definition as file via `-f <path>`.
- TQL files typically end with `.tql`.
- The pipeline may read stdin as data, based on the first operator.
- The pipeline may produce data on stdout, based on the last operator.

### Run integration tests

Build `tenzir` first when testing local changes:

```sh
scripts/build.sh
uvx tenzir-test --root test
```

Common `tenzir-test` options:

- `--passthrough`: Stream output to the terminal (skips reference comparison)
- `--update`: Update reference outputs (check correctness before or after)
- `--debug`: Show detailed test information
- `--match`: Filter test paths with substring or glob matching

### Run unit tests

Run unit tests using the `tenzir-unit-test` binary:

```sh
tenzir-unit-test
```

### Update documentation

User-facing documentation lives in the git-ignored `.docs/` directory, which is
an optional clone of the `tenzir/content` repository.

When changing existing behavior or adding user-facing functionality, update
`.docs/website/src/content/docs/`, create a topic branch there, and open a
companion PR against `tenzir/content`.

Skip this process for internal refactorings that do not affect the user-facing
TQL surface or command line tools.

## C++ development

Read the relevant references below before writing, reviewing, or reasoning
about C++ code. Do not assume patterns from surrounding code—older code may
deviate from current conventions.

### Core principle

Tenzir processes data in columns via Apache Arrow. Evaluate expressions per
series or slice first, then iterate only when row-wise access is necessary.

### Style

- [coding-conventions.md](.agents/references/coding-conventions.md): Formatting, structure, and idioms
- [naming-conventions.md](.agents/references/naming-conventions.md): Naming patterns

### APIs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tenzir/tenzir](https://github.com/tenzir/tenzir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

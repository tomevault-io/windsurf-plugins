---
trigger: always_on
description: You are an expert contributor to microsoft/vscode-cmake-tools, a TypeScript VS Code extension targeting Windows, macOS, and Linux. You are deeply familiar with CMake, CMake Presets, CTest, CPack, generator types (Ninja, Ninja Multi-Config, Visual Studio, Unix Makefiles), kit/toolchain selection, the VS Code extension API, and this repo's architecture. Match existing patterns precisely and always prefer tracing the canonical data flow over guessing or grepping.
---


# CMake Tools — Contributor Instructions

## Domain knowledge

- **Two operating modes**: *presets mode* (`CMakePresets.json`/`CMakeUserPresets.json`) vs. *kits/variants mode*. Many bugs affect only one. Always check `CMakeProject.useCMakePresets` and handle both unless explicitly justified.
- **Kits**: Define compiler, optional toolchain file, optional Visual Studio installation, and environment. On Windows, MSVC kits require VS Developer Environment (`vcvarsall.bat`) merged via `getEffectiveSubprocessEnvironment()` in `cmakeDriver.ts`.
- **Generators**: Single-config (Ninja, Unix Makefiles) use `CMAKE_BUILD_TYPE` at configure time. Multi-config (Ninja Multi-Config, Visual Studio) use `--config` at build time. Never assume single-config.
- **Presets**: `CMakePresets.json` is project-owned (committed). `CMakeUserPresets.json` is user-owned (gitignored). Both support `include` chaining. The merged tree lives in `PresetsController` — never re-parse preset files directly. Types live in `src/presets/preset.ts`.
- **CTest / CPack / Workflow**: Separate drivers — `CTestDriver` (`src/ctest.ts`), `CPackDriver` (`src/cpack.ts`), and `WorkflowDriver` (`src/workflow.ts`) — each with their own preset type.
- **Code model**: The CMake file-API produces `CodeModelContent` (defined in `src/drivers/codeModel.ts`) after configure — the authoritative source for targets, file groups, and toolchains. Never infer targets from `CMakeLists.txt`.
- **Variable expansion**: `src/expand.ts` handles `${variable}` expansion for both kit-context and preset-context vars. Changes here need unit tests.
- **Cross-platform**: Runs on Windows, macOS, Linux. Path separators, env var casing, compiler locations, and generator availability all differ.

## Project conventions

- **Path alias**: `@cmt/*` maps to `src/*` (see `tsconfig.json`). Always use `import foo from '@cmt/foo'` — never relative paths from outside `src/`. `@test/*` maps to `test/*` (defined in both `tsconfig.json` and `test.tsconfig.json`). Use `import ... from '@test/...'` in test files.
- **Error reporting**: Use `rollbar` wrappers for top-level error boundaries around event handlers, never bare `try/catch` that silently swallows.
  - `rollbar.invoke(what, fn)` — wraps synchronous code; catches, logs, and re-throws.
  - `rollbar.invokeAsync(what, fn)` — wraps async function execution; catches promise rejections.
  - `rollbar.takePromise(what, additional, thenable)` — attaches an error handler to an **already-created** Thenable (e.g., a return value from a VS Code API call). Use this when you have a promise but didn't create it via `invokeAsync`.
- **Telemetry**: Use helpers in `src/telemetry.ts` (`logEvent`). Never call the VS Code telemetry API directly.

## Architecture

| Layer | Primary files | Responsibility |
|---|---|---|
| **CMake driver** | `src/drivers/cmakeDriver.ts` (base), `cmakeFileApiDriver.ts`, `cmakeLegacyDriver.ts`, `cmakeServerDriver.ts` | Spawning CMake, file-API replies, code model, cache, targets. `cmakeFileApiDriver` is the modern default. |
| **CMake project** | `src/cmakeProject.ts` | Per-folder state: kit/preset, configure/build/test lifecycle |
| **Build runner** | `src/cmakeBuildRunner.ts` | Build-process orchestration and output streaming |
| **Task provider** | `src/cmakeTaskProvider.ts` | VS Code task integration (`tasks.json` "cmake" type) |
| **Project controller** | `src/projectController.ts` | Multi-folder workspace, active-project routing |
| **Presets** | `src/presets/presetsController.ts`, `presetsParser.ts`, `preset.ts` | Loading, merging, expanding, watching preset files; type definitions |
| **Kits** | `src/kits/kitsController.ts`, `src/kits/kit.ts`, `src/kits/variant.ts` | Compiler scanning, toolchain environment, VS kit detection, variant handling |
| **Extension entry** | `src/extension.ts` | Activation, command registration, wiring all layers |
| **UI / tree views** | `src/ui/` (`projectStatus.ts`, `projectOutline/`, `cacheView.ts`, `pinnedCommands.ts`) | Sidebar views, status bar, context menus |
| **Diagnostics** | `src/diagnostics/` (`cmake.ts`, `build.ts`, `gcc.ts`, `msvc.ts`, `gnu-ld.ts`, etc.) | Output parsing, log-level routing, problem matchers — one file per compiler family |
| **CMake debugger** | `src/debug/cmakeDebugger/` | Debug adapter for CMake script/configure debugging |
| **Language services** | `src/languageServices/` | CMake-language hover, completion, validation |
| **Config** | `src/config.ts` | `ConfigurationReader` — canonical access to all extension settings |
| **Tests** | `test/unit-tests/`, `test/integration-tests/`, `test/end-to-end-tests/`, `test/smoke/` | Mocha suites at four levels of granularity |

## Mandatory rules — apply to every task

### Before touching any code, orient first


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/vscode-cmake-tools](https://github.com/microsoft/vscode-cmake-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

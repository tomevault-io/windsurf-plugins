---
trigger: always_on
description: The nearest ancestor directory (from current working directory upward) that
---

# Concepts

## Root Directory

The nearest ancestor directory (from current working directory upward) that
contains `CMakeLists.txt`. Many commands fail immediately if this directory cannot
be found.

## Local Configuration

`.vim-cmake-naive-config.json` file at the [Root Directory](#root-directory).

Keys:

1. `output` - build output root (default fallback: `build`)
2. `preset` - CMake preset name (empty string means "no preset")
3. `build` - CMake build type (default fallback: `Debug`)
4. `target` - active selected target (missing means `all`)

## Local Cache

`.vim-cmake-naive-cache.json` file at the [Root Directory](#root-directory).

Keys:

1. `targets` - list of discovered target names

## Build Directory

Resolved from [Local Configuration](#local-configuration):

1. If `preset` is empty: `<root>/<output>`
2. If `preset` is set: `<root>/<output>/<preset>`

## Integration State Files

These files are maintained in the [Root Directory](#root-directory):

1. `.vim-cmake-naive-target` - current target name (is empty when target key is
   missing)
2. `.vim-cmake-naive-output` - current build directory path relative to root
   (`<output>` or `<output>/<preset>`)

They are updated whenever local configuration is written.

## Active and Target Compile Commands

1. Active file: `<output>/compile_commands.json` (used as the main compile
   database)
2. Target-local files:
   `<build-directory>/**/CMakeFiles/<target>.dir/compile_commands.json`

## Command Lock

All public `:CMake*` commands run through a single lock.  If one command is
already running, a new command is rejected with: `CMake: another command
<CommandName> is already running`.

## Terminal Reuse

The following commands are run asynchronously and share the same Vim terminal
window:

1. `CMakeBuild`
2. `CMakeTest`
3. `CMakeRun`

# Commands

## CMakeConfig

1. Resolves [Root Directory](#root-directory) from current working directory.
2. Resolves [Local Configuration](#local-configuration) at that root.
3. Creates `.vim-cmake-naive-config.json` only when it does not already exist.
4. Initial payload is an empty JSON object (`{}`), not default values.
5. Updates integration state files (`.vim-cmake-naive-target`,
   `.vim-cmake-naive-output`) immediately after writing config.
6. If config already exists, command does not overwrite it.

## CMakeConfigDefault

1. Resolves [Root Directory](#root-directory).
2. Resolves [Local Configuration](#local-configuration) at that root.
3. Creates config when missing, or updates existing config.
4. Writes default keys:
   1. `output = "build"`
   2. `preset = ""`
   3. `build = "Debug"`
5. Preserves unrelated custom keys in existing config.
6. Updates integration state files.

## CMakeConfigSetOutput `<value>`

1. Requires local configuration to exist.
2. Validates that `<value>` is non-empty.
3. Resolves [Local Configuration](#local-configuration) in current directory or
   parent directories.
4. Sets `output` to the exact provided string.
5. Updates integration state files, including recalculated
   `.vim-cmake-naive-output` based on current `preset`.

## CMakeSwitchPreset

1. Requires local configuration to exist.
2. Resolves [Root Directory](#root-directory), then reads `CMakePresets.json`.
3. Resolves [Local Configuration](#local-configuration).
4. Builds selectable preset list from `configurePresets`:
   1. skips hidden presets
   2. evaluates supported preset conditions (`const`, `equals`, `notequals`,
      `inList`, `notInList`, `matches`, `notMatches`, `anyOf`, `allOf`, `not`)
   3. resolves inheritance and rejects cycles/missing parents
5. Prepends synthetic option `(none)`.
6. Shows [Preset Selection Popup](#preset-selection-popup) when popup support
   exists; otherwise uses menu/inputlist fallback.
7. On selection:
   1. `(none)` removes `preset`
   2. any preset name sets `preset` and removes `build`

## CMakeSwitchBuild

1. Requires local configuration to exist.
2. Resolves [Local Configuration](#local-configuration).
3. Uses current [Local Configuration](#local-configuration) to resolve current
   build value.
4. Presents options: `(none)`, `Debug`, `Release`, `RelWithDebInfo`, `MinSizeRel`.
5. Shows [Build Selection Popup](#build-selection-popup) when popup support
   exists; otherwise uses menu/inputlist fallback.
6. On selection:
   1. `(none)` removes `build`
   2. build name sets `build` and removes `preset`

## CMakeSwitchTarget

1. Requires local configuration to exist.
2. Requires [Local Cache](#local-cache) with `targets` array (usually produced by
   `:CMakeGenerate`).
3. If cache file is missing, reports built-in error: `No cache found. Please run
   CMakeGenerate command first.`
4. Resolves [Local Configuration](#local-configuration).
5. Resolves [Build Directory](#build-directory) and [Scan
   Directory](#scan-directory) from config.
6. Prepends synthetic option `(all)` to cached targets.
7. Shows [Target Selection Popup](#target-selection-popup) when popup support
   exists; otherwise inputlist fallback.
8. On selection:
   1. `(all)`:
      1. copies root `<scan-directory>/compile_commands.json` to
         `<output>/compile_commands.json`
      2. removes `target` key from config
   2. specific target:
      1. resolves `.../CMakeFiles/<target>.dir`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OlegKarasik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

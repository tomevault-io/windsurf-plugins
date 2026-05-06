---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains a macOS sandbox solution to launch applications in a protected environment. The home directory is locked down – only the explicitly provided working directory is accessible. Supports VSCode, terminal shells, Claude Code CLI, and arbitrary commands.

## Files

- **`src/index.ts`** — Main entry point. Orchestrates config loading, sandbox profile generation, and launches the target app via `sandbox-exec`.
- **`src/config.ts`** — App configuration: TOML config loading, built-in app definitions, auto-discovery via `mdfind`.
- **`src/args.ts`** — CLI argument parsing with dynamic mode validation.
- **`src/modes.ts`** — Command building for all modes (shell builtins + configured apps).
- **`src/profile.ts`** — Sandbox profile generation: home scanning, blocklist assembly, SBPL output.
- **`src/guards.ts`** — Safety checks: sandbox nesting, external sandbox detection, workdir validation, app-already-running.
- **`src/drytree.ts`** — `--dry` tree output: visual display of protected/accessible paths.
- **`src/help.ts`** — `--help` output.
- **`src/fmt.ts`** — Terminal formatting helpers (colors, icons, labels).
- **`bxconfig.example.toml`** — Example config with all built-in apps and common extras.
- **`rolldown.config.ts`** — Rolldown bundler config. Builds `dist/bx.js` (ESM, Node shebang).
- **`dist/bx.js`** — Built CLI entry point (generated, not committed).

## Build

```bash
pnpm install
pnpm build        # rolldown → dist/bx.js
pnpm link -g      # install "bx" command globally
```

## Usage

```bash
bx [workdir]                                # VSCode (default mode)
bx code [workdir]                           # VSCode (explicit)
bx xcode [workdir] [-- project-or-workspace] # Xcode
bx term [workdir]                           # sandboxed login shell
bx claude [workdir]                         # Claude Code CLI
bx exec [workdir] -- command [args...]      # arbitrary command

# Custom apps from ~/.bxconfig.toml become modes automatically:
bx cursor [workdir] [-- app-args...]       # if configured
bx zed [workdir]                            # if configured

# Options (work with all modes)
bx --dry ~/work/my-project                  # show what will be protected
bx --verbose term ~/work/my-project         # print generated .sb profile
bx --background code ~/work/my-project      # run in background, log to /tmp
bx --vscode-user code ~/work/my-project      # isolated app profile (default path)
bx --vscode-user ~/my-profile code ~/work/my-project # isolated app profile (custom path)
bx xcode ~/work/my-ios-app -- MyApp.xcworkspace # sandbox dir + explicit open target
```

### Configuration files

**`~/.bxconfig.toml`** — App definitions (TOML format). Each `[<name>]` section becomes a mode usable as `bx <name> [workdir...]`. Built-in apps (`code`, `xcode`) are always available and can be overridden here.

```toml
# Override built-in app path
[code]
path = "/usr/local/bin/code"

# Add a new app (auto-discovered via bundle ID)
# --no-sandbox is auto-detected for Electron apps
[cursor]
bundle = "com.todesktop.230313mzl4w4u92"
binary = "Contents/MacOS/Cursor"

# Add a new app (explicit path, no discovery)
[zed]
path = "/Applications/Zed.app/Contents/MacOS/zed"

# Workdir shortcut — inherits everything from "code"
[myproject]
mode = "code"
paths = ["~/work/my-project", "~/work/shared-lib"]
```

Available fields per app:

| Field | Description |
| --- | --- |
| `mode` | Inherit from another app definition (e.g. `"code"`, `"cursor"`) |
| `path` | Explicit absolute path to the executable (highest priority) |
| `bundle` | macOS bundle identifier for auto-discovery via `mdfind` |
| `binary` | Relative path to executable inside the `.app` bundle |
| `fallback` | Absolute fallback path if discovery fails |
| `args` | Extra arguments always passed to the app |
| `passPaths` | Paths passed as launch args (`true`/`false`/`N`/`["~/p1", "~/p2"]`) |
| `paths` | Default working directories when none given on CLI (supports `~/` paths and `*` globs) |
| `background` | Run the app detached in the background, output to log file (`true`/`false`) |
| `profile` | Use an isolated app profile (`true` = default `~/.vscode-sandbox`, `"path"` = custom path) |

App resolution order: `path` (explicit) → `bundle` + `binary` (mdfind auto-discovery) → `fallback` (hardcoded). See `bxconfig.example.toml` for all options.

When overriding a built-in app, only the fields you specify are replaced — the rest (e.g. `bundle`, `args`) are kept from the built-in definition. When using `mode`, all fields from the referenced app are inherited; own fields override inherited ones.

**`~/.bxignore`** — Unified sandbox rules (paths relative to `$HOME`). One entry per line, empty lines and `#` comments are ignored:

```gitignore
# Block sensitive paths (default, no prefix)
.aws
.kube
.config/sensitive-app

# Allow read-write access to additional directories
rw:work/bin
rw:shared/libs

# Allow read-only access (can read but not modify)
ro:reference/docs
ro:shared/toolchain

# Override built-in protections (files allowed, not just dirs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [holtwick/bx-mac](https://github.com/holtwick/bx-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

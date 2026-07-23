---
trigger: always_on
description: **Never use `npx`.** It is considered dangerous because it can silently fetch and execute arbitrary packages from the registry. Always run binaries through one of these safer mechanisms instead:
---

# Electron Development Guide

## Running node_modules binaries

**Never use `npx`.** It is considered dangerous because it can silently fetch and execute arbitrary packages from the registry. Always run binaries through one of these safer mechanisms instead:

1. **Preferred** — spawn the executable directly from `node_modules/.bin/<tool>` (or the platform equivalent on Windows). This is what `script/lint.js` does for `oxlint`.
2. **Acceptable** — invoke via `yarn <tool>` or `yarn run <tool>`, which resolves to the locally installed version without the registry fallback that `npx` performs.

This rule applies to shell commands you run yourself and to any scripts you author or modify in this repo.

## Project Overview

Electron is a framework for building cross-platform desktop applications using web technologies. It embeds Chromium for rendering and Node.js for backend functionality.

## Directory Structure

```text
electron/                 # This repo (run `e` commands here)
├── shell/               # Core C++ application code
│   ├── browser/         # Main process implementation (107+ API modules)
│   ├── renderer/        # Renderer process code
│   ├── common/          # Shared code between processes
│   ├── app/             # Application entry points
│   └── services/        # Node.js service integration
├── lib/                 # TypeScript/JavaScript library code
│   ├── browser/         # Main process JS (47 API implementations)
│   ├── renderer/        # Renderer process JS
│   └── common/          # Shared JS modules
├── patches/             # Patches for upstream dependencies
│   ├── chromium/        # ~159 patches to Chromium
│   ├── node/            # ~48 patches to Node.js
│   └── ...              # Other targets (v8, boringssl, etc.)
├── spec/                # Test suite (1189+ TypeScript test files)
├── docs/                # API documentation and guides
├── build/               # Build configuration
├── script/              # Build and automation scripts
└── chromium_src/        # Chromium source overrides
../                      # Parent directory is Chromium source
```

## Build Tools Setup

Electron uses `@electron/build-tools` for development. The `e` command is the primary CLI.

**Installation:**

```bash
npm i -g @electron/build-tools
```

**Configuration location:** `~/.electron_build_tools/configs/`

## Essential Commands

### Configuration Management

| Command | Purpose |
|---------|---------|
| `e init <name> --root=<path> --bootstrap testing` | Create new build config and sync |
| `e use <name>` | Switch to a different build configuration |
| `e show current` | Display active configuration name |
| `e show configs` | List all available configurations |

### Build & Development Loop

| Command | Purpose |
|---------|---------|
| `e sync` | Fetch/update all source code and apply patches |
| `e sync --3` | Sync with 3-way merge (required for Chromium upgrades) |
| `e build` | Build Electron (runs GN + Ninja) |
| `e build -k 999` | Build and continue on errors (up to 999) |
| `e build -t <target>` | Build specific target (e.g., `electron:node_headers`) |
| `e start` | Run the built Electron executable |
| `e start --version` | Verify Electron launches and print version |
| `e test` | Run the test suite |
| `e debug` | Run Electron in debugger (lldb on macOS, gdb on Linux) |

### Patch Management

| Command | Purpose |
|---------|---------|
| `e patches <target>` | Export patches for a target (chromium, node, v8, etc.) |
| `e patches all` | Export all patches from all targets |
| `e patches --list-targets` | List available patch targets |

## Typical Development Workflow

```bash
# 1. Ensure you're on the right config
e show current

# 2. Sync to get latest code
e sync

# 3. Make your changes in shell/ or lib/ or ../

# 4. Build
e build

# 5. Test your changes (Leave the user to do this, don't run these commands unless asked)
e start
e test

# 6. If you modified patched files in Chromium:
cd ..  # Go to Chromium repo
git add <files>
git commit -m "description of change"
cd electron
e patches chromium  # Export the patch
```

## Patches System

Electron patches upstream dependencies (Chromium, Node.js, V8, etc.) to add features or modify behavior.

**How patches work:**

```text
patches/{target}/*.patch  →  [e sync --3]  →  target repo commits
                          ←  [e patches]   ←
```

**Patch configuration:** `patches/config.json` maps patch directories to target repos.

**Key rules:**

- Fix existing patches 99% of the time rather than creating new ones
- Preserve original authorship in TODO comments
- Never change TODO assignees (`TODO(name)` must retain original name)
- Each patch file includes commit message explaining its purpose

**Creating/modifying patches:**

1. Make changes in the target repo (e.g., `../` for Chromium)
2. Create a git commit
3. Run `e patches <target>` to export

**Fixing patch conflicts on an existing PR:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [electron/electron](https://github.com/electron/electron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

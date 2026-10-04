---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Comfy Chair is a single-binary Go CLI (Charm `huh`/`lipgloss` TUI + `fsnotify`) for managing ComfyUI installations and developing custom nodes: lifecycle control (start/stop/restart/update/install), node scaffolding from templates, live-reload on file changes, and node packaging. It shells out to `git`, `uv`, and the ComfyUI venv's Python; it does not embed ComfyUI.

## Build / run / test

```bash
task build          # go build -o comfy-chair .  (preferred)
task build-dev      # build with debug symbols (-gcflags="all=-N -l")
task build-all      # cross-compile linux/darwin/windows amd64 into dist/
task run            # run ./comfy-chair
task install        # go install into $GOBIN
go build -o comfy-chair .   # direct build, no Taskfile

go vet ./...        # vet
gofmt -l .          # check formatting (code must be gofmt-clean)
```

Run tests with `go test ./...` (placeholder rendering in `nodes_test.go`, torch-install/sysinfo tests in `internal/`, and an httptest end-to-end suite in `internal/assets/`). Releases are cut by `.github/workflows/release.yml` via GoReleaser (linux/windows/darwin amd64, `CGO_ENABLED=0`).

Bump `AppVersion` in `internal/constants.go` when releasing.

## Architecture

Three Go packages:

- **root `package main`** — `main.go` (CLI entry, ComfyUI lifecycle, install, env management, migrations, asset-server command), `nodes.go` (node CRUD/scaffolding/packing), `reload.go` (fsnotify watcher + debounced restart), `procattr_unix.go` / `procattr_windows.go` (platform process attributes via build tags).
- **`internal/`** — shared modules: `cli.go` (CLIRouter), `core.go` (active-install resolution + env-confirmation wrapper), `menu.go` (interactive TUI), `install.go` (torch/GPU install, node requirements), `process.go`, `pidfile.go`, `health.go`, `performance.go`, `sysinfo_unix.go`/`sysinfo_windows.go`, `utils.go` (config registry, state-file resolution), `logger.go`, `constants.go`.
- **`internal/assets/`** — the asset manager web server (see below); decoupled from `internal`.

### Command dispatch flow (in `main()`)

1. `initPaths()` resolves config/paths.
2. `internal.NewCLIRouter(...)` + `router.SetupCLICommands(...)` registers command handlers — handlers are defined in `main.go` and **injected into the internal router as function values** (e.g. `startComfyUI`, `createNewNode`). Standalone commands (`assets`, `empty-trash`) are registered with `router.RegisterCommand` directly. `internal` owns routing/help/flags, `main` owns the implementations.
3. `router.Route(os.Args)` handles the command (unknown commands exit inside `Route`); with no command the **interactive TUI menu** (`internal/menu.go`) launches.

When adding a command, wire it in three places consistently: the handler in `main.go`, registration in `SetupCLICommands` (or `RegisterCommand` + the help-category list in `ShowHelp`), and (if user-facing) the menu via `MenuChoices`. Command names support both `snake_case` and `kebab-case` aliases.

### Multi-environment model (important)

Comfy Chair manages **multiple named ComfyUI installs** (`lounge`, `den`, `nook`), persisted in **`comfy-installs.json`** (path, type, `is_default`, `custom_nodes`, and `reload_include_dirs`). Lifecycle commands take a `*internal.ComfyInstall` parameter and run through `internal.RunWithEnvConfirmation(action, fn)`, which resolves/prompts for the target environment and passes it to the handler — handlers must act on the passed install, never re-resolve. `WORKING_COMFY_ENV` in `.env` pins the active environment.

### Configuration

- State files (`.env`, `comfy-installs.json`, `comfy-performance-history.json`) live in **`os.UserConfigDir()/comfy-chair/`** (e.g. `~/.config/comfy-chair/`), resolved via `internal.ResolveStateFile`, which one-time copy-migrates files from the legacy binary-adjacent location. Pid/log files (`comfyui.pid`, `comfyui.log`) live in the ComfyUI install directory.
- **`.env`** (loaded via `godotenv`): `COMFYUI_PATH` (required), `COMFY_RELOAD_EXTS`, `COMFY_RELOAD_DEBOUNCE`, `COMFY_START_FLAGS`, `COMFY_FRONTEND_VERSION`, `GPU_TYPE`, `PYTHON_VERSION`, `TORCH_INSTALL_CMD_NVIDIA`, `CUSTOM_NODES_AUTHOR`, `CUSTOM_NODES_PUBID`, `WORKING_COMFY_ENV`. Missing required vars trigger interactive setup. Copy from `.env.example`.
- **`comfy-installs.json`**: the multi-environment registry described above.
- Paths in config support portable placeholders `{HOME}` / `{USERPROFILE}` (resolved in `internal.ExpandUserPath`).

### venv detection (by design constraint)

Only venvs named exactly **`venv`** or **`.venv`** inside the ComfyUI dir are recognized (`FindVenvPython`). Custom venv names are intentionally unsupported — do not add support for them. Python deps are managed with **`uv`**, and PyTorch install is GPU-specific (`GPU_TYPE`).

### Node scaffolding


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [regiellis/comfy-chair-go](https://github.com/regiellis/comfy-chair-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->

---
trigger: always_on
description: Instructions for coding agents using macbox or working in this repository.
---

# Agent instructions for macbox

Instructions for coding agents using macbox or working in this repository.

## Purpose

macbox smoke-tests macOS applications inside disposable local VMs. It returns structured JSON with the launch result, screenshot path, logs, crash reports, and a report artifact with the verdict, diagnosis, and next actions. The host stays clean.

## When to use macbox

Use macbox when the user wants to:

- Verify a macOS `.app`, `.dmg`, or `.pkg` launches in a fresh environment
- Capture crash reports or logs after launch
- Test a build without installing it on the host
- Validate installer flows and release gates
- Run an isolated macOS check from an IDE via MCP

Do not use macbox for:

- Linux or iOS builds
- Cloud CI farms or multi-tenant hosting
- Reading arbitrary host files
- Bypassing code signing, Gatekeeper, or Keychain

## Prerequisites (user-side)

The user must have completed one-time setup:

- Tart installed
- Template VM `macos-sequoia-clean` (or configured `default_image`)
- SSH key at `~/.ssh/macbox_id` installed in guest `admin` account
- `macbox doctor --json` returns `"ok": true`

If doctor fails, point the user to [docs/GUIDE.md](docs/GUIDE.md). Do not guess at VM credentials in macbox config.

## MCP profiles (preferred for app testing)

Configure `macbox-core` for routine app testing so agents do not carry every advanced guest-control tool in every turn. Add `macbox-power` only when the workflow needs warm VMs, installers, release gates, guest scripts, semantic UI inspection, keyboard/mouse automation, or arbitrary guest file transfer.

- `mcp/macbox_core_mcp.py` exposes the core smoke-test/lifecycle surface.
- `mcp/macbox_power_mcp.py` exposes the advanced guest-control surface.
- `mcp/macbox_mcp.py` remains backward-compatible and exposes the full surface. It also honors `MACBOX_MCP_PROFILE=core|power|all`.

### Core MCP tools

| Tool | Use |
|------|-----|
| `macbox_status` | Check readiness before starting |
| `list_images` | List available local Tart images |
| `list_profiles` | List macbox sandbox profiles |
| `create_sandbox` | New disposable VM from template |
| `watch_sandbox` | Return native window/VNC watch instructions for a running VM |
| `upload_app` | Copy `.app` bundle to guest |
| `upload_dmg` | Copy `.dmg` to guest |
| `upload_pkg` | Copy `.pkg` to guest |
| `run_app_smoke_test` | Launch + wait + collect evidence |
| `collect_logs` | Syslog excerpt |
| `take_screenshot` | PNG capture |
| `collect_crashes` | DiagnosticReports |
| `get_run_report` | Fetch the structured report for a run |
| `reset_sandbox` | Reset a disposable VM |
| `stop_sandbox` | Stop a VM without deleting it |
| `run_doctor` | Run environment readiness checks |
| `destroy_sandbox` | Required cleanup |

### Power MCP tools

| Tool | Use |
|------|-----|
| `create_warm_sandbox` | Start a reusable warm VM |
| `run_on_warm_sandbox` | Upload a local `.app` to a warm VM and run it |
| `exec_in_guest` | Run a guest shell command inside the VM |
| `run_applescript_in_guest` | Run guest AppleScript for UI automation or inspection |
| `run_jxa_in_guest` | Run guest JavaScript for Automation (ObjC/Quartz bridge) |
| `prepare_agent_workspace` | Create/reset guest workspace directories for agent scripts |
| `run_script_in_guest` | Run shell/AppleScript/JXA from a guest-side file and save diagnostics |
| `observe_guest` | Screenshot + frontmost app/window + screen/windows/process state |
| `inspect_ui_tree` | Inspect guest Accessibility tree |
| `click_ui_element` | Press a UI element by role/title/name selector |
| `type_text_in_guest` | Type literal text into the frontmost guest app |
| `paste_text_in_guest` | Paste text through the guest clipboard |
| `send_keys_in_guest` | Send a key or key-combination (named keys + modifiers) |
| `click_in_guest` | Click at guest screen coordinates |
| `scroll_in_guest` | Send a guest scroll event |
| `drag_in_guest` | Drag between guest screen coordinates |
| `open_guest_app` | Launch a guest app with optional arguments |
| `list_guest_windows` | Inspect current guest window titles |
| `list_guest_processes` | Inspect guest process state |
| `push_file_to_guest` | Copy any file or directory to a guest path |
| `pull_file_from_guest` | Download any file or directory from the guest |
| `mount_dmg_image` | Mount a DMG in guest |
| `install_dmg_guest_app` | Copy an app from a mounted DMG into `/Applications` |
| `install_guest_pkg` | Run installer validation and optionally launch the installed app |
| `run_installed_guest_app` | Launch app from `/Applications` |
| `assert_window` | Verify window title content |
| `assert_app_running` | Verify running bundle ID |
| `run_release_gate` | One-shot pass/fail validation |
| `run_release_matrix` | Fan an artifact across multiple images |
| `reset_warm_sandbox` | Reset a warm VM back to clean state |

Most smoke tests only need `macbox-core`.

## Standard workflow

1. Call `macbox_status()`. Stop if not ready.
2. `create_sandbox(image="macos-sequoia-clean", display_mode="headless")`
3. `upload_app(vm_name, "/absolute/path/to/App.app")`
4. `run_app_smoke_test(vm_name, "App.app", timeout_seconds=120)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robzilla1738/macbox](https://github.com/robzilla1738/macbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->

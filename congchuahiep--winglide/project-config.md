---
trigger: always_on
description: Rust (edition 2021) app for **Windows 11 only**, version 0.0.9. Single binary (`WinGlide.exe`), no workspace/monorepo. Features:
---

# AGENTS.md - WinGlide

## Project summary

Rust (edition 2021) app for **Windows 11 only**, version 0.0.9. Single binary (`WinGlide.exe`), no workspace/monorepo. Features:

- Cycle through taskbar buttons via global hotkeys (`Alt+[` / `Alt+]`)
- Uncombine taskbar buttons (unique AUMID per window)
- On-screen virtual desktop indicator (drawn on the taskbar)
- Jump to virtual desktop via `Alt+1`..`Alt+9`
- System tray icon + Settings GUI + auto-update check

Uses IUIAutomation because the Win11 taskbar is XAML, not Win32 HWNDs.

## Developer commands

```bash
cargo build --release              # binary -> target/release/WinGlide.exe
cargo build                        # debug build
cargo check                        # quick type-check (no codegen needed)
cargo run --release                # run
cargo run -- --debug --verbose     # run with debug console + verbose logging
./target/release/WinGlide.exe            # normal run
./target/release/WinGlide.exe -v         # with debug logging
./target/release/WinGlide.exe --settings-ui   # only open the Settings GUI
```

No lint/formatter config exists in the repo - only `cargo check` / `cargo build` are available.

## CLI args (manual parsing in cli.rs, no clap)

- `-v` / `--verbose` - enable debug-level logging
- `--debug` - attach/alloc console for debug logging (also enables console worker)
- `--console-worker` - run as standalone debug console worker process
- `--settings-ui` - launch only the Settings GUI (XAML)
- `--reopen-ui` - after starting the background app, reopen the Settings UI

`Args.combine_enabled` field exists but is never set by any flag (dead).

## Architecture

```
main.rs                     -> panic hook -> dispatch: cli::parse_args -> bootstrap (single-instance, DPI, debug console) -> mode routing
cli.rs                      -> manual arg parsing (RunMode: ConsoleWorker / SettingsUi / BackgroundApp)
config.rs                   -> AppConfig: serde JSON at %APPDATA%/WinGlide/config.json (see "Config")
bootstrap.rs                -> ensure_single_instance (named mutex), attach_debug_console, setup_dpi_awareness
app.rs                      -> orchestrator: wires hotkey_manager + key_hook + enumerator + uncombine_manager + tray + indicator + hidden window
hotkey/
├── mod.rs                  -> module doc + re-export: HotkeyManager, HotkeyAction, LowLevelKeyHook
├── manager.rs              -> HotkeyManager: RegisterHotKey + dispatch classification; HotkeyAction::CycleLeft/CycleRight/SwitchVirtualDesktop(idx)
└── low_level_hook.rs       -> generic WH_KEYBOARD_LL hook: overrides Windows-owned combos (any Win+key); posts WM_HOTKEY with the same IDs
taskbar/
├── mod.rs                  -> module doc + re-export: TaskbarEnumerator, CycleDirection, UncombineManager
├── enumerator.rs           -> IUIAutomation: enumerate buttons, 1s TTL cache, cycle_to_neighbor
├── button_window.rs        -> ButtonWindowMap: map button ↔ window (AUMID -> PID -> Title -> Process)
└── uncombine.rs            -> UncombineManager: sets unique AppUserModelID per window
win32/
├── mod.rs                  -> re-exports win32 submodules
├── window.rs               -> EnumWindows: find_visible_windows, get_process_name
├── activate.rs             -> force_activate (SetForegroundWindow + AttachThreadInput)
├── aumid.rs                -> get/window AUMID helpers (SHGetPropertyStoreForWindow)
├── explorer.rs             -> get_explorer_pid, invalidate_explorer_pid_cache
└── window_context.rs       -> WindowContext::current_state(): foreground window + monitor + virtual desktop
event/
├── mod.rs                  -> re-exports; defines WM_APP_RELOAD_CONFIG (0x102), WM_APP_RESTART_AS_ADMIN (0x103)
├── uia.rs                  -> UIA StructureChanged hook -> WM_APP_INVALIDATE_CACHE (0x101)
└── winevent.rs             -> WinEvent EVENT_OBJECT_SHOW hook -> WM_APP_UNCOMBINE (0x100)
virtual_desktop/
└── indicator.rs            -> IndicatorWindow: layered window drawing desktop dots on the taskbar (winvd); left-click = switch desktop, right-click = "Move to Desktop" context menu, Alt+click = move foreground window there; placement (Auto/Left/Right) is user-configurable via Settings
tray_icon.rs                -> Shell_NotifyIconW tray icon + context menu (Exit / Settings / Debug Console)
setting/                    -> windows-reactor native GUI settings (hotkey capture, toggles, update check)
logging/                    -> tracing-subscriber: rolling file + detached console via named pipes; tracing-forest format
admin.rs                    -> is_running_as_admin, restart_as_admin (ShellExecuteW "runas")
autostart.rs                -> HKCU\...\Run registry autostart enable/disable
updater.rs                  -> GitHub Releases API check (reqwest blocking), finds .msi asset
types.rs                    -> shared data structs (TaskbarButton, WindowInfo, TargetWindow), no logic, no imports
utils.rs                    -> clean_button_name, truncate, is_system_class, is_light_theme
```

## Config

`AppConfig` is serialized to `%APPDATA%/WinGlide/config.json` (via `dirs::config_dir()`). Loaded at startup in `main.rs`, reloaded by `WM_APP_RELOAD_CONFIG`. Fields:

- `uncombine_mode: bool` (default true)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [congchuahiep/WinGlide](https://github.com/congchuahiep/WinGlide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

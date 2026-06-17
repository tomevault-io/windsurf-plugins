---
trigger: always_on
description: validates_stdout_format/parses_bool/parses_verbose, profile_unset
---

# AGENTS Session History

## Build Command
```powershell
cargo build
```

## Test Command
```powershell
cargo test --workspace --lib
```

## Clippy Command
```powershell
cargo clippy --workspace --lib --bins -- -D warnings
```

## Test Counts (as of last run)
- **Total**: 1,510 tests across 9 crates (all pass)
- **overthrone-core**: 639
- **overthrone-pilot**: 117
- **overthrone-forge**: 84
- **overthrone-hunter**: 76
- **overthrone-crawler**: 97
- **overthrone-reaper**: 178
- **overthrone-relay**: 100
- **overthrone-scribe**: 54
- **overthrone-viewer**: 25
- **overthrone-cli bin**: 128 (+14 profile tests in cli_config, +31 profile subcommand tests in commands::config, +25 cli_config tests, +14 commands::config tests, +2 wizard arg tests, +12 session tests inside)

## Completed Tasks (Honest Account)

### From S-Rank Plan (this session — CLI completion: 5 tasks)

18. **cli #2 — Config file loading (TOML, XDG-style)** (DONE):
    - `cli_config.rs` expanded to 1111 lines with full CRUD operations
    - `CliConfig` struct: 17 fields with `Serialize`/`Deserialize`
    - Functions: `load_config()`, `save_config()`, `set_value()`, `unset_value()`, `display()` (masks secrets)
    - XDG-aware path resolution: `$XDG_CONFIG_HOME/overthrone/config.toml` (Linux), `%APPDATA%\overthrone\config.toml` (Windows)
    - Precedence: CLI flag > env var > config file > built-in default
    - `CONFIG_KEYS` registry (17 keys) with per-key validation
    - 39 unit tests: parse_minimal/parse_full/parse_unknown_keys_ignored, save_then_load_roundtrip, save_creates_parent_dir, set/unset value, validation of auth_method/stdout_format/verbose/bool parsing, display masks secrets, mask_secret short strings, default_config_path
    - `ovt config` subcommand with 8 actions: `init`, `show`, `path`, `set`, `unset`, `edit`, `save`, `profile`

19. **cli #3 — Profile system** (DONE):
    - Named profiles at `<config_dir>/profiles/<NAME>.toml`
    - Honor `OT_CONFIG` and `OT_PROFILE` env vars for custom config/profile locations
    - Functions: `load_profile()`, `save_profile()`, `delete_profile()`, `list_profiles()`, `clone_profile()`, `validate_profile_name()`, `active_profile()`, `profile_exists()`, `default_profiles_dir()`
    - Profile name validation: rejects path traversal, >64 chars, control chars, path separators
    - 14 unit tests: name validation (accept/reject/path-traversal/too-long/invalid-chars), profile path, save/load roundtrip, load returns default when missing, active_profile reads env, list_profiles empty dir, delete rejects missing, profile_exists rejects invalid, clone roundtrip
    - `ovt config profile` subcommand with 9 actions: `list`, `show`, `create`, `set`, `unset`, `delete`, `use`, `clone`, `path`
    - 31 new tests in `commands::config`: profile_create_refuse/force/rejects_invalid_name, profile_list, profile_show_missing/existing/requires_name, profile_delete_missing/removes_existing, profile_use_prints/warns, profile_clone_copies/rejects_existing/rejects_invalid, profile_path_resolvable/requires_env, profile_set creates/replaces/rejects_unknown_key/validates_auth_method/validates_stdout_format/parses_bool/parses_verbose, profile_unset clears/rejects_unknown/errors_when_missing/handles_all_field_types, profile_action_dispatch_handles_all_variants, profile_path_for rejects_invalid, current_profiles_dir_is_under_config_dir
    - Config merge refactored to `apply_config_layer()` helper: CLI flag > env > active profile > main config > default

20. **cli #5 — TUI audit complete** (DONE):
    - Verified 6 TUI modules: `app`, `event`, `graph_view`, `runner`, `ui`, `mod`
    - All wired to `ovt tui` command via `cmd_tui()` in `commands_impl.rs`
    - Supports two modes: live crawler (with credentials) and view-only (graph display)
    - Graph loading from JSON files via `AttackGraph::from_json_file()`
    - `graph_view.rs` (1741 lines): Full attack graph visualization with node/edge rendering
    - `runner.rs`: Terminal setup with crossterm, 30 FPS rendering loop
    - `app.rs`: Application state management with tabs
    - `ui.rs`: Layout and widget rendering
    - `event.rs`: Keyboard/mouse event handling

21. **cli #4 — Interactive shell mode for forge** (DONE):
    - `interactive_shell.rs` (3263 lines): Full REPL with rustyline
    - Features: tab completion, command history, syntax highlighting, validator
    - Forge modules: `forge/golden`, `forge/silver`, `forge/diamond`, `forge/skeleton`
    - Module system: `use <module>`, `set <option> <value>`, `unset <option>`, `run`
    - Additional commands: help, connect, disconnect, exec, enum, kerberos, smb, graph, reaper, exit, quit, clear, history, whoami, hostname, pwd, ls, cd, upload, download, info, sessions, bg, fg, log, spawn, pivot, migrate, steal_token, rev2self, getuid, getpid, ps
    - Wired to `ovt shell` command via `cmd_shell()` in `commands_impl.rs`
    - Supports WinRM, SMB, WMI shell types via `ShellType` enum
    - Module context tracking with required options validation

### From S-Rank Plan (previous session — 13 tasks ticked off)

1. **relay #1 — LDAPS TLS wrapping** (DONE):
   - Added `RelayStreamType` enum (Plain/Tls variant) to `relay.rs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Karmanya03/Overthrone](https://github.com/Karmanya03/Overthrone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

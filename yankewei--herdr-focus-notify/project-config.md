---
trigger: always_on
description: A Rust CLI binary that runs as a **Herdr plugin** on macOS. It listens for Herdr's `pane.agent_status_changed` event and emits clickable macOS desktop notifications via `alerter`. Clicking a notification focuses the matching Herdr agent pane.
---

# Agent Notes for `herdr-focus-notify`

## Project Type

A Rust CLI binary that runs as a **Herdr plugin** on macOS. It listens for Herdr's `pane.agent_status_changed` event and emits clickable macOS desktop notifications via `alerter`. Clicking a notification focuses the matching Herdr agent pane.

The plugin manifest is in [`herdr-plugin.toml`](herdr-plugin.toml). The binary is built by Herdr itself using the command declared in that manifest.

## Essential Commands

| Command | Purpose |
|---|---|
| `cargo build --release` | Build the release binary to `target/release/herdr-focus-notify`. This is exactly what the plugin manifest uses. |
| `cargo test` | Run unit and integration tests. |
| `cargo fmt -- --check` | Check formatting. |
| `cargo clippy --all-targets --all-features -- -D warnings` | Lint; CI treats warnings as errors. |
| `herdr plugin link .` | Install the plugin locally from the repo root. |
| `target/release/herdr-focus-notify --test` | Trigger a test notification manually (declared as an action in `herdr-plugin.toml`). |

CI runs on `macos-latest` and executes all of the above in order (see [`.github/workflows/ci.yml`](.github/workflows/ci.yml)).

## Project Structure

```
.
├── Cargo.toml          # Rust package metadata; minimal dependencies (serde, serde_json)
├── herdr-plugin.toml   # Herdr plugin manifest: build command, actions, event subscriptions
├── src/main.rs         # Thin CLI/plugin entry point
├── src/*.rs            # Focused modules for CLI, config, event parsing, focus checks, scripts, and notifier delivery
├── assets/icons        # Bundled local agent icons used by alerter --app-icon
├── tests/cli_test.rs   # Process-level CLI contract tests
├── .env.example        # Documented configuration template for plugin users
├── README.md           # English documentation
└── README.zh-CN.md     # Chinese documentation
```

There are no submodules, no external crates beyond serde/serde_json, and no build scripts.

## Architecture and Control Flow

1. **Entry point**: `main()` calls `run()`, prints real errors to stderr, and returns a non-zero exit code.
2. **Event source**:
   - In normal mode, the binary reads the Herdr event from the `HERDR_PLUGIN_EVENT_JSON` environment variable.
   - In test mode (`--test` CLI arg), it fabricates a notification for the currently focused pane. When the plugin is enabled, test mode always sends: it bypasses the status filter and downgrades a `Skip` focus decision to a plain send (no visibility monitor), so the action can validate the pipeline even in a fully configured setup.
   - `--help` and `--version` print to stdout before plugin setup.
3. **Notification decision**:
   - Only `blocked` and `done` statuses can produce notifications (they are the ones that need user action); `HERDR_FOCUS_NOTIFY_STATUSES` can only narrow that set, never extend it.
   - The notification is skipped when the target pane is already focused **and** the frontmost macOS application is one of the effective terminal bundle IDs (`effective_terminal_bundle_ids()`: explicit `ACTIVATE_APP` targets first, then the terminal bound to the pane's workspace). Any other or unknown frontmost app sends the notification to avoid missing a state change; when a terminal is known, the send becomes a visibility-monitored send that auto-dismisses once the pane is seen.
   - `pane.focused` events bind the frontmost terminal to the pane's workspace (`learn_terminal_from_frontmost`), so the plugin works with zero configuration; this is what makes `ACTIVATE_APP` optional.
   - Recognized agent names are matched to bundled local PNG icons and passed to `alerter` with `--app-icon`.
   - Notification titles and bodies use short status-specific copy: blocked agents ask the user to review and respond, while done agents ask the user to review the result. The plugin does not read or summarize pane contents.
4. **Binary resolution**:
   - `herdr` is resolved from `HERDR_BIN_PATH`, then `PATH`, then hard-coded candidates (`~/.local/bin/herdr`, `/opt/homebrew/bin/herdr`, `/usr/local/bin/herdr`), defaulting to `"herdr"`.
   - The notifier backend is resolved from `HERDR_FOCUS_NOTIFY_NOTIFIER`, then `PATH`, then hard-coded candidates for `alerter`.
5. **Focus script generation**:
   - A shell script is written to `HERDR_PLUGIN_STATE_DIR` (falling back to `$TMPDIR/herdr-focus-notify`).
   - The script name is a hash of the pane ID, so repeated events for one pane reuse the same script path. Old generated scripts and crashed notifier temp files are cleaned up opportunistically.
   - The script is made executable with mode `0o700`.
6. **Notification delivery**:
   - Normal plugin events spawn the script detached via `nohup sh ... &`. The script itself calls `alerter`, then activates the terminal (configured via `ACTIVATE_APP`, or `open -b <learned bundle id>` when unconfigured) and runs `herdr agent focus <pane>` if the user clicks the notification.
   - `--test` runs the generated script in the foreground so notifier failures surface through stderr and a non-zero exit code.

## Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yankewei/herdr-focus-notify](https://github.com/yankewei/herdr-focus-notify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->

---
trigger: always_on
description: This repository is Shovel: a native desktop database client built as a Rust workspace with Dioxus Desktop 0.7.3.
---

This repository is Shovel: a native desktop database client built as a Rust workspace with Dioxus Desktop 0.7.3.

Use this file as the repo-specific operating manual. Prefer it over generic framework advice.

## Core rules

- This project uses Dioxus 0.7 APIs only. `cx`, `Scope`, and `use_state` do not belong here.
- The app is desktop-first. Do not assume web routing, SSR, or fullstack patterns unless the user explicitly asks for them.
- Prefer existing workspace patterns over generic Dioxus examples.
- Keep code examples concise, but anchor decisions in the actual repo structure.
- Before changing behavior, inspect the relevant crate. The workspace is modular, but the `ui` crate still orchestrates a lot directly.

## Product summary

Shovel is a desktop DB client with:

- SQLite, PostgreSQL, MySQL, and ClickHouse connections
- schema explorer, table previews, query execution, exports, and row editing
- saved queries and query history
- persistent chat threads and ACP-assisted SQL workflows
- ACP registry agent install/connect flows plus an embedded Ollama ACP bridge
- dark/light theming and persisted UI settings

Per the current README, ClickHouse supports connect/explore/query/export, but not table-row editing.

## Workspace map

The root `Cargo.toml` defines a multi-crate workspace. The important crates are:

- `app`: desktop shell, Dioxus launch, crash reporting, window config, embedded ACP entrypoint
- `ui`: Dioxus UI, global app state, settings modal, workspace screen, connect screen
- `models`: shared domain models and persisted settings types
- `storage`: local persistence for settings, sessions, saved connections, query history, saved queries, chat DB
- `connection`: DB connection orchestration and SSH tunnel lifecycle
- `connection-ssh`: SSH tunnel implementation
- `database`: common driver traits and error types
- `driver-sqlite`, `driver-postgres`, `driver-mysql`, `driver-clickhouse`: backend-specific drivers
- `drivers`: shared driver-facing types
- `explorer`: schema/database tree loading and table metadata
- `query-core`: core query execution and table-edit logic
- `query-format`: SQL formatting
- `query-io`: import/export logic
- `query`: facade re-exporting `query-core`, `query-format`, and `query-io`
- `acp`: ACP runtime, stdio bridge, permissions, terminals, embedded Ollama support
- `acp-registry`: ACP registry fetch/install logic
- `services`: facade crate re-exporting major operations, but currently not used as a real boundary by the rest of the workspace

## Runtime and startup flow

- `app/src/main.rs` is the real entrypoint.
- Normal startup installs a panic hook, configures the Dioxus desktop window, injects `app.css`, and launches `ui::App`.
- Panic handling writes crash logs under the OS temp dir in `shovel/crash-<timestamp>.log` and shows a native error dialog.
- On Unix desktop builds the app sets `app_id = "dev.shovel.app"`.
- The binary also supports an embedded ACP mode:
  - `shovel acp-agent ollama --model ...`
  - this bypasses UI launch and runs the embedded ACP agent instead

## UI architecture

The UI is Dioxus 0.7 and currently uses modern hooks correctly:

- `use_signal`
- `use_resource`
- `use_effect`
- `#[component]`

Important state facts:

- `ui/src/app.rs` loads persisted UI settings and SQL format settings via `use_resource`.
- It restores connection sessions on launch when `restore_session_on_launch` is enabled.
- It persists updated UI settings and SQL format settings back to storage from effects.
- `ui/src/app_state.rs` contains global signals for:
  - `APP_STATE`
  - theme
  - UI settings
  - SQL format settings
  - settings modal visibility
  - history visibility
  - tooltips
  - toast notifications
- `app_state` also owns session add/remove/restore behavior and triggers session persistence.

This means UI work often spans both component files and `app_state.rs`. Do not assume state is localized.

## Persisted UI settings

`models::AppUiSettings` currently persists:

- `theme`
- `ai_features_enabled`
- `restore_session_on_launch`
- `show_saved_queries`
- `show_connections`
- `show_explorer`
- `show_history`
- `show_sql_editor`
- `show_agent_panel`
- `default_page_size`
- `tool_panel_layout`

Important current-tree fact:

- There is no separate `show_query_manager` flag in the current codebase.
- `show_saved_queries` exists and is persisted.

When adding a new persisted UI toggle, update all of these together:

- `models/src/settings.rs` defaults and serde-compat tests
- any settings modal controls
- workspace visibility/filter helpers
- toolbar buttons or toggle entrypoints
- any flows that should auto-open the relevant panel

## Storage model

Local app data lives under:

- `dirs::data_local_dir()/shovel`

Current storage files include:

- `saved_connections.json`
- `session_state.json`
- `saved_queries.json`
- `query_history.json`
- `sql_format_settings.json`
- `app_ui_settings.json`
- `shovel.db`
- `acp/workspace/...`

Storage behavior that matters:

- Saved connection metadata is serialized without passwords.
- Passwords are stored via `keyring` service `shovel.connections`.
- Secret entries use a hashed key derived from `request.identity_key()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fynth/Shovel](https://github.com/Fynth/Shovel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->

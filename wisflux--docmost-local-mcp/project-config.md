---
trigger: always_on
description: Guidance for working in this repository.
---

# CLAUDE.md

Guidance for working in this repository.

## Response style

- **Explain simply.** Answer in plain, concise language. Avoid jargon where a plain word works; when a technical term is unavoidable, define it in one short clause.
 
- **Lead with the diagram or a one-line summary**, then add brief supporting text — not a wall of prose.

## What this is

`docmost-local-mcp` is a **Rust, read-only MCP server** that fronts a self-hosted [Docmost](https://docmost.com) instance for local IDE / AI tools (Cursor, Claude Desktop, etc.). It speaks MCP over **stdio** using [`rmcp`](https://docs.rs/rmcp) 0.6, authenticates to Docmost with email/password (storing the session locally), and exposes documentation as MCP tools. It is distributed as an `npx` package (`@wisflux/docmost-local-mcp`) whose Node launcher downloads the Rust binary from GitHub Releases.

Edition **2024**. The crate is both a binary and a library (`docmost_local_mcp`); integration tests in `tests/` consume the library.

## Commands

```bash
cargo build                 # debug build (default feature: native-webview = tao + wry)
cargo build --release       # release build
cargo test                  # unit + integration tests (tests/ use tempfile + mock axum servers)
cargo fmt --check                                        # format — CI runs this
cargo clippy --all-targets --all-features -- -D warnings  # lint — CI fails on any warning

# Run the MCP server locally (talks MCP over stdio):
cargo run -- --base-url=https://docs.example.com
DOCMOST_BASE_URL=https://docs.example.com cargo run

# Build without the native auth window (forces browser-fallback login):
cargo build --no-default-features
```

- `--base-url` (or `DOCMOST_BASE_URL`) selects the Docmost instance; it's optional — if absent the interactive login asks for it.
- `DOCMOST_DISABLE_KEYRING=1` skips the OS keyring and uses the encrypted-file credential fallback (used by tests).
- `DEBUG_DOCMOST_MCP=1` (or `true`) enables debug logging (via `debug::debug_log`, prefix `[docmost-local-mcp][ts][scope]`), which goes to **stderr** — never stdout, which is reserved for the MCP protocol.

## Architecture

Request path: **MCP client → `DocmostMcpServer` (`#[tool_router]`) → `DocmostClient` → Docmost `/api/...` → Markdown string back to the client.**

- [src/main.rs](src/main.rs) — `clap` CLI. Default command builds `StartupConfig` and serves over `stdio()`. Hidden `auth-window` subcommand launches the webview helper.
- [src/server.rs](src/server.rs) + [src/server/tools.rs](src/server/tools.rs) + [src/server/tools_write.rs](src/server/tools_write.rs) — `DocmostMcpServer { client, tool_router }`, `#[tool_handler] impl ServerHandler`. **20 tools**: 10 read-only (`list_spaces`, `search_docs`, `search_pages` [alias], `get_space`, `get_page`, `list_pages`, `list_child_pages`, `get_comments`, `list_workspace_members`, `get_current_user`, all `read_only_hint = true`) plus 10 **write** tools (`read_only_hint = false`): `create_page`/`update_page` (Markdown → ProseMirror) in `tools.rs`, and `duplicate_page`, `copy_page_to_space`, `move_page`, `move_page_to_space`, `create_space`, `update_space`, `create_comment`, `update_comment` in `tools_write.rs`. The write tools use a separate `#[tool_router(router = write_tool_router)]` merged into the main router in `new()` (`tool_router() + write_tool_router()`), keeping each tools file within the size limit.
- [src/server/render.rs](src/server/render.rs) — formats domain structs into the **Markdown** strings tools return (results are truncated: search 5, lists 10, members 20).
- [src/prosemirror/](src/prosemirror/) — Markdown ⇄ ProseMirror conversion, split into `reader.rs` (JSON → Markdown), `writer/` (Markdown → JSON, event-walker in `writer/build.rs`), and `nodes.rs` (node builders). Mentions use a link convention: `[label](user:UUID)` / `[label](page:UUID)` → a `mention` node (inline atom with `entityType`/`entityId`; each gets a unique `id` since Docmost dedups by it). Comments accept a StarterKit subset — no tables/task-lists/images.
- [src/docmost_client.rs](src/docmost_client.rs) — `reqwest` wrapper. Every call is `POST {base_url}{endpoint}` with `bearer_auth(token)`; responses are unwrapped from an `{ "data": ... }` envelope. List shapes normalized by `normalize_list_result` / `normalize_cursor_list_result`. **Retries once on HTTP 401** after reauthenticating.
- [src/auth/](src/auth/) — `manager.rs` (session lifecycle: reuse saved session unless within 2 min of JWT expiry, else reauth via saved credentials or interactive login), `local_server.rs` (axum login page on `127.0.0.1:<random>`), `webview.rs` (native `tao`/`wry` window with browser fallback).
- [src/storage/](src/storage/) — `state_store.rs` persists to `~/.docmost-local-mcp/` (`config.json`, `session.json`); credentials go to the **OS keyring first** (`keyring_store.rs`), falling back to an **AES-256-GCM** encrypted file. Writes are atomic (temp + rename) with `0o600` perms.
- [src/types/](src/types/) — `mod.rs` holds the serde domain models (`#[serde(rename_all = "camelCase")]`); `inputs.rs` holds the `JsonSchema` tool-input structs (re-exported, so `crate::types::*Input` paths are unchanged).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wisflux/docmost-local-mcp](https://github.com/wisflux/docmost-local-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

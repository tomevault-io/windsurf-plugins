---
trigger: always_on
description: A production-grade MCP server for Godot. Python server (FastMCP v3) communicates over WebSocket with a GDScript editor plugin. AI clients call MCP tools → Python routes commands → Godot plugin executes against the editor API → results flow back.
---

# CLAUDE.md — Godot AI

## What this project is

A production-grade MCP server for Godot. Python server (FastMCP v3) communicates over WebSocket with a GDScript editor plugin. AI clients call MCP tools → Python routes commands → Godot plugin executes against the editor API → results flow back.

## Architecture

```
AI Client → MCP (stdio/sse/streamable-http) → Python FastMCP server → WebSocket (port 9500) → Godot EditorPlugin
```

- **Python server**: `src/godot_ai/` — FastMCP v3, async, lifespan manages WebSocket server
- **GDScript plugin**: `plugin/addons/godot_ai/` — canonical source; symlinked into `test_project/addons/` for testing
- **Protocol**: JSON over WebSocket. Request/response with `request_id` correlation. Handshake on connect.
- **Session model**: Multiple Godot editors can connect. Tools route through active session.
- **Handler/Runtime layer**: Shared handlers in `src/godot_ai/handlers/` contain tool logic. They depend on a `Runtime` protocol (`runtime/interface.py`), implemented by `DirectRuntime` for the in-process server. Tools and resources are thin wrappers that create a runtime and delegate.
- **Readiness gating**: Write operations check session readiness (`ready`/`importing`/`playing`/`no_scene`) before executing. Plugin sends readiness in handshake and via `readiness_changed` events. Python `require_writable()` in `handlers/_readiness.py` gates all write handlers.

## Key conventions

- **GDScript plugin is the canonical copy** in `plugin/`. `test_project/addons/godot_ai` is a symlink — no copy needed.
- **Error codes**: Defined in `protocol/errors.py` (Python) and `utils/error_codes.gd` (GDScript). Keep in sync. Use Godot's built-in `error_string(err)` to translate numeric error codes in error messages — do not write a custom lookup table.
- **Tools return `dict`**: Handlers call `runtime.send_command(command, params)` which returns a dict or raises. Tools create a `DirectRuntime` and delegate to handlers.
- **Plugin runs on main thread**: All GDScript executes in `_process()` with a 4ms frame budget. Never block. Use `call_deferred` for scene tree mutations.
- **Scene paths are clean**: `/Main/Camera3D` format, not raw Godot internal paths. Use `ScenePath.from_node(node, scene_root)` in GDScript.
- **MCP logging**: Plugin prints `MCP | [recv] command(params)` / `MCP | [send] command -> ok` to Godot console. Controlled by `mcp_logging` var.
- **Tool-search-friendly naming**: All MCP tools use `domain_action` namespacing (`scene_*`, `node_*`, `script_*`, etc.). Non-core tools are tagged `meta={"defer_loading": True}` for Anthropic tool-search compatibility; core tools (`editor_state`, `scene_get_hierarchy`, `node_get_properties`, `session_list`, `session_activate`) stay non-deferred. Plugin command names (sent over WebSocket) are independent — the MCP tool `editor_reload_plugin` dispatches the plugin command `reload_plugin`.
- **`batch_execute` uses plugin command names, not MCP tool names**: The MCP tool `node_create` dispatches the plugin command `create_node`. Inside `batch_execute`'s `commands[].command` field, use the plugin name (`create_node`), not the MCP name (`node_create`). The Python handlers in `src/godot_ai/handlers/` are the authoritative map — each handler calls `runtime.send_command("<plugin_cmd>", ...)`. When an agent passes the wrong form, the error message spells out the convention and suggests near-matches from `dispatcher.suggest_similar()`, with structured fuzzy matches in `error.data.suggestions`.
- **Session IDs**: format is `<project-slug>@<4hex>` (e.g. `godot-ai@a3f2`). The slug is derived from the project directory name so agents can recognize which editor they're targeting; the hex suffix disambiguates same-project twins. Server treats the ID as an opaque key.
- **Per-call session routing**: every Godot-talking tool accepts an optional `session_id` parameter. Empty (the default) resolves to the global active session. When supplied, that single call targets that session — `require_writable` and every handler inside the call see the pinned session, not the active one. Use this when multiple AI clients share one MCP server. Resources (`godot://...`) still resolve via the active session.

## Worktrees

Claude Code sessions often run in git worktrees (`.claude/worktrees/<name>/`). Be aware of which worktree you're in — it affects everything:

- **File paths**: Your working directory is the worktree, not the repo root. Files you create live in that worktree.
- **Godot editor**: The editor runs against a specific worktree's `test_project/`. The plugin is symlinked from that worktree's `plugin/` directory. Check `session_list` — the `project_path` field tells you which worktree the editor is using.
- **Dev server**: The `--reload` dev server uses the root repo's `.venv` and `src/`, not the worktree's. Python code changes in any worktree won't take effect unless the root repo also has them (e.g. via `git pull`). To serve the worktree's own Python source, use `script/serve-this-worktree` (prepends the worktree's `src/` to `PYTHONPATH` and frees port 8000 before starting).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hi-godot/godot-ai](https://github.com/hi-godot/godot-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

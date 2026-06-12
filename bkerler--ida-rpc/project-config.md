---
trigger: always_on
description: Agent-focused guidance for developing and extending the IDA Pro RPC plugin.
---

# AGENTS.md — ida-rpc

Agent-focused guidance for developing and extending the IDA Pro RPC plugin.

## Automatic Reverse Engineering Usage

When reversing binaries with IDA Pro, prefer `ida-rpc` before screen scraping,
manual GUI steps, or ad hoc IDAPython scripts.

Use these stable agent probes first:

```bash
ida-rpc capabilities
ida-rpc find-project <binary-or-idb>
```

Then use:

```bash
ida-rpc open <binary-or-idb> --headless --detach
ida-rpc metadata --project <idb>
ida-rpc functions --project <idb> --limit 50
ida-rpc decompile <function-or-address> --project <idb>
ida-rpc xrefs-to <target> --project <idb>
ida-rpc xrefs-from <target> --project <idb>
ida-rpc rename-function <target> <new_name> --project <idb>
ida-rpc set-comment <address> <comment> --project <idb>
ida-rpc save --project <idb>
```

All automation commands print JSON to stdout. Treat `{"ok": false, ...}` as a
recoverable tool error. Set `IDA_RPC_DEBUG=1` only when a traceback is needed.

RPC queries should always be run outside the sandbox.

## Project Overview

`ida-rpc` is a JSON-RPC daemon that runs **inside IDA Pro** (as a plugin) and exposes reverse-engineering capabilities over a Unix domain socket. It is wire-format compatible with [ghidra-rpc](https://github.com/cellebrite-labs/ghidra-rpc).

```
┌─────────────┐      Unix Socket       ┌──────────────────────────────┐
│  LLM agent  │  ──── JSON/newline ──→ │  ida-rpc daemon              │
│  (via CLI)  │  ←── JSON/newline ───  │  (IDA Python plugin + server)│
└─────────────┘                        └──────────────────────────────┘
```

## Repository Layout

| Path | Purpose |
|------|---------|
| `ida_rpc_plugin.py` | IDA Pro plugin entry point (loaded by IDA on startup) |
| `ida_rpc/cli.py` | Click-based CLI (`ida-rpc` command) |
| `ida_rpc/client.py` | Unix socket client + error types |
| `ida_rpc/daemon.py` | Daemon lifecycle (`start`, `stop`, `restart`) |
| `ida_rpc/session.py` | Session persistence (JSON state files) |
| `ida_rpc/server/main.py` | JSON-line socket server, handler registry |
| `ida_rpc/server/context.py` | `IdaContext` — wraps IDA database, thread-safe dispatch |
| `ida_rpc/server/tools/` | One module per command family (see below) |

### Tools Modules

| Module | Commands |
|--------|----------|
| `analysis.py` | `load`, `list_binaries`, `functions`, `imports`, `exports`, `metadata`, `relocations`, `list_calling_conventions`, `save` |
| `basefind.py` | `basefind` |
| `decompiler.py` | `decompile`, `decompile_all` |
| `disassembly.py` | `disassemble` |
| `assembler.py` | `assemble` |
| `search.py` | `find_bytes`, `strings`, `symbols` |
| `memory.py` | `read_bytes`, `write_bytes`, `memory_map` |
| `segments.py` | `add_segment`, `edit_segment`, `delete_segment`, `list_segments` |
| `xrefs.py` | `xrefs_to`, `xrefs_from` |
| `navigation.py` | `goto` (GUI only) |
| `modifications.py` | `rename_function`, `rename_symbol`, `create_label`, `set_comment`, `set_function_signature`, `set_data_type`, `create_function`, `delete_function`, `create_instruction`, `undefine`, `set_thunk`, `set_calling_convention`, `batch_rename`, `batch_set_comment` |
| `data_types.py` | `create_struct`, `create_union`, `create_enum`, `list_data_types`, `list_labels`, `modify_struct`, `modify_enum`, `clear_data_range`, `apply_data_type_range`, `set_equate`, `list_equates` |
| `bookmarks.py` | `set_bookmark`, `list_bookmarks`, `remove_bookmark` |
| `cfg.py` | `basic_blocks` |
| `tags.py` | `tag_function`, `untag_function`, `list_tags`, `functions_by_tag` |
| `processor.py` | `get_processor_context`, `set_processor_context` |
| `namespaces.py` | `create_namespace`, `list_namespaces` |

## Development Environment

- **IDA Pro Version:** 9.3 SP2 at `$(IDA_INSTALL_DIR)` (e.g. `/home/bjk/bin/ida-pro-9.3sp2`)
- **Plugin install path:** `$IDAUSR/plugins/ida_rpc_plugin.py` (default `$IDAUSR` is `~/.idapro/` on Linux, `~/Library/Application Support/IDA Pro/` on macOS, `%APPDATA%\Hex-Rays\IDA Pro\` on Windows)
- **Plugin symlink (dev):** `ln -s /path/to/ida-rpc/ida_rpc_plugin.py $(IDA_INSTALL_DIR)/plugins/ida_rpc_plugin.py`
- **Python package:** `pip install -e /path/to/ida-rpc`
- **Entry points:** `ida-rpc` (CLI), `ida-rpcd` (daemon)

### Installing / Updating

```bash
# Production install via the IDA Plugin Manager
hcli plugin install ida-rpc

# Development install (after code changes, reinstall so the symlinked plugin sees updates)
pip install -e /path/to/ida-rpc

# Or use uv
uv pip install -e /path/to/ida-rpc
```

## Adding a New Command

1. **Pick or create a tools module** in `ida_rpc/server/tools/`. Group by semantic area.
2. **Write the handler function:**

```python
from ida_rpc.server.main import register_handler

def _handle_my_command(ctx, args: dict) -> dict:
    # ctx: IdaContext — provides resolve_address(), find_function(), run_on_main_thread()
    # args: dict of command arguments from the JSON request
    # Must return a dict (will be JSON-serialized)
    result = {"foo": "bar"}
    return result

register_handler("my_command", _handle_my_command)
```

3. **Import the module** in `ida_rpc/server/tools/__init__.py`:

```python
from ida_rpc.server.tools import my_module
```

4. **Add CLI passthrough** in `ida_rpc/cli.py`:

```python
@cli.command(name="my-command")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bkerler/ida_rpc](https://github.com/bkerler/ida_rpc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->

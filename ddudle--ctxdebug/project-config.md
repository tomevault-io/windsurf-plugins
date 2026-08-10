---
trigger: always_on
description: MCP server platform that connects WinDbg, IDA Pro 9.x, and x64dbg to Claude AI for AI-assisted reverse engineering and Windows security research.
---

# MCO — Multi-debugger Controller Orchestrator

MCP server platform that connects WinDbg, IDA Pro 9.x, and x64dbg to Claude AI for AI-assisted reverse engineering and Windows security research.

**Owner:** Windows security researcher (heap exploitation, anti-debug).
**Language policy:** Code and docs in English; user communicates in Russian (informal).

---

## Repository layout

```
C:\Users\dange\mcp\mco\
├── windbg_mcp.py            # WinDbg via cdb.exe subprocess. 70+ tools. PRODUCTION.
├── ida_mcp.py               # IDA Pro 9.x via HTTP REST (localhost:2022). 32+ tools.
├── ida_server_plugin.py     # Paste into IDA Python console to start HTTP server.
├── agent/                   # x64dbg MCP via named pipe
│   ├── __main__.py          # Entry: python -m agent --mcp
│   ├── core.py              # Shared debugging context (bridge, skills, state)
│   ├── bridge.py            # Named-pipe IPC to x64dbg C++ plugin
│   ├── mcp_server.py        # MCP server + 35+ tool definitions
│   └── plugins/
│       ├── x64dbg_plugin.cpp   # C++ plugin — compile with build_plugin.bat
│       └── build_plugin.bat    # MSVC build (produces mco_agent.dp64)
├── mco_orchestrator.py      # Cross-debugger meta-tools. 7 compound workflows.
├── mco_sessions.py          # SQLite session recorder/replayer. 13 tools.
├── mcp_config_example.json  # All MCP server configs with env vars
├── pyproject.toml           # Python package (x64dbg-mcp-agent)
├── sessions.db              # SQLite DB (auto-created by mco_sessions.py)
├── site/index.html          # Startup landing page
└── .windbg_mcp_breakpoints/ # Saved breakpoint sets (cmd files)
```

---

## MCP servers — quick reference

| Server name    | File                   | Provides                                    | Tool count |
|----------------|------------------------|---------------------------------------------|------------|
| `windbg`       | `windbg_mcp.py`        | Crash dumps, heap, shadow stack, kernel     | 70+        |
| `ida`          | `ida_mcp.py`           | Static analysis, decompile, rename, patches | 32+        |
| `x64dbg`       | `agent/__main__.py`    | Dynamic analysis, anti-debug bypass        | 35+        |
| `mco`          | `mco_orchestrator.py`  | Cross-debugger compound workflows           | 7          |
| `mco-sessions` | `mco_sessions.py`      | Session recording, FTS search, export      | 13         |

---

## Adding servers to Claude Code

```powershell
# Run from any directory
claude mcp add windbg     -- python "C:\Users\dange\mcp\mco\windbg_mcp.py"
claude mcp add ida        -- python "C:\Users\dange\mcp\mco\ida_mcp.py"
claude mcp add mco        -- python "C:\Users\dange\mcp\mco\mco_orchestrator.py"
claude mcp add mco-sessions -- python "C:\Users\dange\mcp\mco\mco_sessions.py"

# x64dbg: cwd MUST be the mco directory
claude mcp add x64dbg -- python -m agent --mcp
# (set cwd to C:\Users\dange\mcp\mco in your MCP client config)
```

Use `mcp_config_example.json` as a reference for full JSON config including env vars.

---

## Pre-flight: what needs to be running

### WinDbg server
- Needs `cdb.exe` at the path set in `CDB_PATH` / `WINDBG_MCP_CDB`
- Default: `C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\cdb.exe`
- No process needed pre-launch — tools open dumps or attach on demand.

### IDA server
- IDA Pro 9.x must be open with a binary loaded.
- Then in IDA's Python console (Alt+F7 or Python tab):
  ```python
  exec(open(r'C:\Users\dange\mcp\mco\ida_server_plugin.py').read())
  ```
- IDA 9.x may auto-start the HTTP server on port 2022; if not, the plugin starts it.
- The client auto-discovers the right endpoint from 6 candidates (`/api/v1/py`, `/api/python`, `/python`, `/exec`, `/api/1/exec`, `/api/v1/python`).

### x64dbg server
- x64dbg must be running with the `mco_agent.dp64` plugin loaded.
- Build the plugin: `cd agent\plugins && build_plugin.bat`
- Copy `mco_agent.dp64` to x64dbg's plugins folder, restart x64dbg.
- The plugin exposes a named pipe at `\\.\pipe\x64dbg_ai_agent`.
- Rebuild required if `bridge.py` protocol changes (magic header `X64A` + 4-byte len + 8 padding bytes).

### Orchestrator / Sessions
- Work standalone — no debugger required to add the servers.
- `mco` makes direct connections to cdb.exe, IDA HTTP, and x64dbg pipe.
- `mco-sessions` only needs SQLite (`sessions.db` in project root).

---

## Environment variables

### WinDbg (`windbg_mcp.py`)
| Variable            | Default                                                              | Purpose                        |
|---------------------|----------------------------------------------------------------------|--------------------------------|
| `WINDBG_MCP_CDB`    | `C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\cdb.exe`     | Path to cdb.exe                |
| `CDB_PATH`          | same default                                                         | Alias used by orchestrator     |

### IDA (`ida_mcp.py`)
| Variable         | Default       | Purpose                              |
|------------------|---------------|--------------------------------------|
| `IDA_MCP_HOST`   | `localhost`   | IDA HTTP server host                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DdUdle/ctxdebug](https://github.com/DdUdle/ctxdebug) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->

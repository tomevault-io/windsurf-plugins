---
trigger: always_on
description: Instructions for coding agents (Claude Code, Codex, Cursor, etc.) working with **co-pymol**.
---

# AGENTS.md

Instructions for coding agents (Claude Code, Codex, Cursor, etc.) working with **co-pymol**.

**What co-pymol is:** a PyMOL plugin that starts an MCP server inside PyMOL's own Python process, exposing the `pymol.cmd` API (plus a gemmi-backed metrics layer for pLDDT/ipTM/pTM/PAE) as tools. Once installed, an MCP client like Claude Code or Cursor can drive PyMOL in natural language.

Two scenarios — jump to whichever fits:

1. **You're editing this repo** → see [§1 Working on this repo](#1-working-on-this-repo).
2. **You're helping a user install co-pymol on their machine** → see [§2 Installing co-pymol on a user's machine](#2-installing-co-pymol-on-a-users-machine).

---

## 1. Working on this repo

### Architecture

- **Plugin runs inside PyMOL's process.** On startup (`__init_plugin__`), an MCP server launches in a daemon background thread on port 8766.
- **MCP server** (`src/co_pymol/server.py`) exposes PyMOL's `cmd` module as MCP tools. MCP clients (Claude Code, Cursor, etc.) connect via `http://localhost:8766/sse`.
- **Metrics** (`src/co_pymol/core/metrics.py`) uses gemmi for structure metadata extraction — not PyMOL. This keeps metric parsing clean and avoids polluting PyMOL's object state. Reads PAE/ipTM/pTM from `_ma_qa_metric_*` categories in mmCIF first, falls back to sibling JSON.
- **Triage** (`src/co_pymol/core/triage.py`) manages navigation/flagging state for reviewing batches of structures (mobile eval workflow).

### Layers

The package uses a **src-layout**: it lives at `src/co_pymol/`. Inside it:

- **package root** (`__init__.py`, `cli.py`, `server.py`) — entry points; `constants.py` holds shared constants (port, palette, etc.); `instructions.py` loads `MCP_INSTRUCTIONS` from the sibling `instructions.md`. No domain logic.
- **`core/`** — domain logic + state, no MCP: `session.py` (per-session state), `metrics.py` and `triage.py` (pure, no PyMOL). `triage_view.py` is the one exception — it drives PyMOL to render a focused structure (`triage_render`); the pure triage state stays in `triage.py`. `session` depends on `metrics`/`triage`.
- **`utils/pymol/`** — cross-cutting PyMOL primitives: `helper.py` (`ensure_pymol`, `pymol_lock`) and `render.py` (`render_image`, `apply_plddt_palette`).
- **`tools/`** — thin MCP wrappers, one `register_*_tools(mcp)` per file; no logic beyond marshalling to `core/` and `utils/`.

### Thread safety

All `pymol.cmd` calls are serialized with `pymol_lock` (a `threading.Lock`). The MCP server runs in a daemon thread; PyMOL's GUI runs on the main thread. Rendering (`cmd.ray`, `cmd.png`) definitely needs the lock. Most read operations work from threads in modern PyMOL, but we lock everything for safety.

### Agent-facing behavior

The MCP server pushes its own instructions (`src/co_pymol/instructions.md`) to every connected client. That file is the right place to change cross-client agent behavior (e.g. "don't auto-render after operations") — not this AGENTS.md, and not per-client config.

### Dev setup

Install into PyMOL's bundled Python (the same rule as the user-facing install — see §2 for the full playbook with troubleshooting):

```bash
/Applications/PyMOL.app/Contents/bin/python -m pip install --user -e ".[dev]"
```

Then:

- **Tests:** `pytest`
- **Pre-commit hooks (optional):** `pre-commit install` — see `.pre-commit-config.yaml`
- **Commit style:** `type: subject` (see `git log` for examples — `refactor:`, `docs:`, `chore:`, `fix:`, etc.)

### How to add new tools

1. Add a `register_*_tools(mcp)` function in the relevant `src/co_pymol/tools/` file (or a new one), then call it from `create_server()` in `src/co_pymol/server.py`
2. Inside the register function, add a new function decorated with `@mcp.tool()`
3. Use `pymol_lock` for any `pymol.cmd` calls
4. Return a string (status message) or `Image` (for rendered output)

```python
@mcp.tool()
def my_new_tool(arg: str) -> str:
    """Description shown to Claude."""
    cmd = ensure_pymol()
    with pymol_lock:
        cmd.some_operation(arg)
        return f"Done: {arg}"
```

### Dependencies

- `mcp~=1.27.1` — official MCP Python SDK; we use its bundled `mcp.server.fastmcp.FastMCP` (no standalone `fastmcp` package). Pinned tight on purpose — MCP is co-pymol's network-facing trust boundary and `FastMCP` has had API churn between minors; bump the pin deliberately, not opportunistically.
- `gemmi>=0.6` — mmCIF/PDB parsing for metrics (atom data + AF3 `_ma_qa_metric_*`)
- `numpy` — array ops for pLDDT/PAE in metrics
- PyMOL — **not a pip dependency**, install the app from pymol.org. Install this plugin into PyMOL's Python: `/Applications/PyMOL.app/Contents/bin/python -m pip install --user -e .`

---

## 2. Installing co-pymol on a user's machine

Because co-pymol lives inside PyMOL, it installs into **PyMOL's bundled Python**, not the system Python or any venv. On macOS that interpreter lives at `/Applications/PyMOL.app/Contents/bin/python`. On Linux/conda installs the path will differ — ask the user for it before running anything. If they're not sure, `which pymol` followed by checking for a sibling `python` in the same `bin/` directory is usually the right interpreter.

### Prerequisites to check


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soo-jeongkim/co-pymol](https://github.com/soo-jeongkim/co-pymol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->

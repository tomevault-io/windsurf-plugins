---
trigger: always_on
description: Context for coding agents modifying this repo. (User-facing "how to use Hunch" lives in
---

# AGENTS.md — working on Hunch

Context for coding agents modifying this repo. (User-facing "how to use Hunch" lives in
`README.md`; this file is "how the code works and how not to break it.")

Hunch drives a real, logged-in Mac for an LLM **focus-free** — reading and acting on
background apps without stealing the user's screen, cursor, or keyboard. That invariant is
the product. Most gotchas below exist to protect it.

## Setup & tests

```bash
.venv/bin/python -m pytest tests/          # full suite; needs pyobjc (the venv has it)
.venv/bin/python -m pytest tests/test_smoke.py
```

- Python ≥ 3.11. Depends on **pyobjc** (AppKit/ApplicationServices/Quartz) — macOS only.
- `tests/test_smoke.py::test_tool_count` asserts **exactly 29 MCP tools**. Adding/removing a
  tool means updating that number in the same commit, on purpose.
- Tests fake all AX/Quartz calls (`monkeypatch`), so they run headless with no UI and touch
  no Keychain. Keep new logic unit-testable this way: put the OS call behind a function you
  can patch, assert on the decision logic.

## Architecture — one engine, three faces

The SDK is the product; everything else is an app built on it.

- **`sdk.py`** — `Hunch`, the developer-facing SDK object (instance-owned policy, auth
  injection `ApiKey`/`OAuthToken`/`"none"`, `app_id` namespacing, notify handler).
- **`server.py`** — the MCP server, **an app built ON the SDK** (the first one). 29
  `@mcp.tool()` functions each call `_run(name, ...)` → `agent._dispatch_core`. Its only
  "personal machine" specialness is constructor args (`policy="personal"`, Hunch branding).
- **`agent.py`** — the agent loop (`Agent.run`), two backends: **api** (`anthropic`, metered)
  and **subscription** (`claude-agent-sdk`, the user's Claude sign-in). `backend="auto"`
  picks by ambient credentials. **`_dispatch_core(mac, name, args)` is the single tool-execution
  engine** shared by both the server and the agent loop — expected Hunch exceptions become
  plain content strings so the model can adapt; only unexpected ones set `is_error`.
- **`local_mac.py`** — the macOS backend: `MacSession` (AX perception + action primitives)
  and `LocalComputer` (exposes snapshot/act/screenshot to the loop). The biggest, most
  delicate file.
- **`ax_tree_mac.py`** — low-level AX helpers (`get_attr`/`get_attrs`, `get_window`,
  `get_actions`, `values_to_bounds`, tree walk). One IPC round-trip matters here; huge trees
  (Mail inbox = one AXRow per email) make naive per-attribute reads take minutes.
- **`cdp.py`** — the web/Electron backend (Chrome DevTools Protocol), background-driven.
- **`gate.py` / `policy.py`** — consent. `Gate.front_gate` / `confirm_dialog`; `confirm="off"`
  or `HUNCH_NO_INTERNAL_GATE=1` env = auto-approve (host owns permissions, e.g. the desktop app).
- `os_ops.py` (files/clipboard/AppleScript), `creds.py`/`auth.py` (Keychain), `notify.py`,
  `errors.py`, `playbook.py` (the MCP instruction string), `cli.py`.

### Adding or changing a tool

A tool touches **three** places — keep them in sync:
1. **`server.py`** — a `@mcp.tool()` function calling `_run("name", ...)`.
2. **`agent.py`** — an entry in `AGENT_TOOLS` (schema) and one in `_DISPATCH` (name → callable).
3. The implementation (usually a `MacSession`/`LocalComputer` method, or `os_ops`).

Then bump `test_tool_count`. Native-AX actions (click/type/window/…) are *not* separate tools;
they're `action` values inside the `act` tool — edit `_ACTION_ITEM` (agent.py) and the `TOOLS`
`act` schema (local_mac.py), which must agree.

## The focus-free contract (do not regress)

Hunch may **never** touch the shared cursor/keyboard or raise an app except as a gated last
resort. Concretely:

- **Prefer the AX layer**: `AXPress` / `AXUIElementSetAttributeValue` trigger elements with no
  cursor movement and no app activation. `snapshot`/`act` operate on background windows.
- **Shared-input fallbacks (pixel click, keystrokes) must `activate()`-or-refuse** — never post
  a CGEvent blind. See `click`/`right_click`/`set_text` in `local_mac.py`: if the AX path fails
  and `allow_pixel`/`allow_keystrokes` is off (simultaneous mode), they return a helpful refusal
  string, not an action.
- **`MacSession.disturbances`** counts every shared-input use (pixel clicks, keystrokes, key
  combos, app raises). `act()` appends a per-call receipt when a call disturbed the screen. If
  you add a shared-input path, increment the right counter — the receipt is how "focus-free" is
  audited in-band.
- Layer priority, most-direct first: **OS-API → AppleScript → Web/CDP → AX → vision**. Vision
  (`screenshot` + `click_xy`) is the gated last resort; don't reach for it when a tree read works.

## AX-layer knowledge (hard-won — read before touching `local_mac.py`)

- **Stable refs**: `snapshot` assigns `[eN]` per element via a keymap; the same element keeps its
  ref across snapshots. `act()` returns a **delta** (`~ changed / + new / gone:`) vs the last
  snapshot, full tree only on first view / window change / >50% churn. Don't assume `act` re-sends
  the whole tree.
- **The `_ax_activate` ladder** (`click` uses it): try the element's own `AXPress` → press-like
  alternative actions (`AXOpen`/`AXConfirm`/…) → a **read-back-verified** `AXValue` flip for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PrithviSeran/hunch-mcp](https://github.com/PrithviSeran/hunch-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->

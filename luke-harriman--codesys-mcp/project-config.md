---
trigger: always_on
description: Orientation for a future Claude / agent session working on this repo. Read first.
---

# CLAUDE.md - codesys-mcp-persistent

Orientation for a future Claude / agent session working on this repo. Read first.

## What this is

`codesys-mcp-persistent` is an MCP server that wraps the **CODESYS V3.5 IDE scripting API** (IronPython 2.7) as MCP tools. Owned/maintained by Luke Harriman (`luke-harriman` on GitHub). Drop-in superset of `@codesys/mcp-toolkit`.

- **Target runtime:** Windows + CODESYS V3.5 SP19 / SP20 + Node 18+. Not portable.
- **SP21+ persistent mode is currently broken.** The watcher's `se.system.execute_on_primary_thread()` call no longer exists in SP21 P5 / SP22 P1 (and possibly earlier patches — SP21 P3 release note `CDS-94322 "Scripting: Context object for Python calls needed"` is the closest paper trail but unconfirmed). Symptom: every persistent-mode tool call returns `Marshal error: The functionality 'system.execute_on_primary_thread(...)' is no longer supported`. Headless mode (`--noUI` per command) is unaffected. The fix is well-understood (single-thread watcher + `se.system.delay()` between polls — see [phobicdotno/Codesys-MCP-SP21-plus](https://github.com/phobicdotno/Codesys-MCP-SP21-plus) v0.4.2 for the reference implementation) but not yet adopted here. See known pitfall #12 and the deferred-work entry.
- **Wire format:** stdio JSON-RPC (MCP) on the outside; file-based IPC on the inside.
- **Two execution modes:** `persistent` (CODESYS UI stays open, watcher.py running, commands marshalled onto its primary thread) and `headless` (`--noUI` spawn-per-command). Persistent is the point of the project; headless is the bootstrap + fallback (and the only SP21+-compatible mode until the watcher rewrite lands).

Read `README.md` for tool surface, `ARCHITECTURE.md` for IPC details, `CHANGELOG.md` for the per-version journey.

## Hard architectural constraints (don't fight these)

1. **IronPython 2.7 inside CODESYS.** Scripts run there, not in CPython. No `f""` strings, no `print()` parens, no `nonlocal`, `unicode` is a separate type from `str`, ints can be `long`, `range()` returns a list. The default `json.dumps(..., ensure_ascii=True)` calls a defective decode path that crashes on cp1252 bytes - **always use `ensure_ascii=False` and emit utf-8 via `_text_utils.emit_result()`**.
2. **ASCII-only Python source** (no `# -*- coding -*-` header in current scripts). Em-dashes, smart quotes, `§` are forbidden in source. The static checker `dev/check-scripts.ps1` enforces this; run it before every commit.
3. **CODESYS scripting API is single-threaded.** Current watcher marshals from a .NET background thread onto the IDE's primary UI thread via `se.system.execute_on_primary_thread()` (src/scripts/watcher.py:230). A long-running script (regex DoS, big bulk read) freezes the IDE. **This API was removed in SP21+** — see pitfall #12. The forward fix runs the watcher directly on the primary thread and yields via `se.system.delay()` between polls; the IDE stays interactive because `system.delay()` services the message loop. Until that rewrite ships, SP21+ users must run with `--mode headless`.
4. **`scriptengine.online` ops need an executor frame on real PLCs.** `create_online_application`, `oa.login`, `oa.start`, `oa.read_value`, `oa.set_prepared_value`, etc. raise `InvalidOperationException: Stack empty.` from a pure IPC-driven script — the IDE-internal `_executionStack` is only populated by the script executor's `Executing` event, which the IPC bridge bypasses. Fixed in 0.6.3 by `ensure_online_connection.py`: reflects into `scriptengine.online._executor` and routes every online call through its public `ExecuteSource(source)` method (see `with_executor`). **When writing a new online tool, wrap each `online_app.X(...)` call in `with_executor(online_app.X, ...args)`** — without it the call hits Stack empty on real PLCs.
5. **`set_pou_code` auto-saves to disk.** UI Ctrl+Z does NOT recover prior content. Read before overwriting; treat the project file as the source of truth, not the IDE buffer.

## How scripts are delivered

Scripts in `src/scripts/*.py` are **templates with `{KEY}` placeholders**. They are NOT imported by Python; they are **concatenated at the TS layer** via `ScriptManager.prepareScriptWithHelpers(name, params, helpers)`:

1. Helper files (e.g. `_text_utils`, `ensure_project_open`, `find_object_by_path`) are loaded and concatenated first.
2. The main script is appended.
3. `interpolate()` runs `{KEY}` substitution with **automatic Python string escaping** (`pyEscape` in `script-manager.ts`). Use `{KEY:raw}` to bypass escaping (rare; only `set_pou_code` declaration/implementation use this).

**Consequence:** helpers and the main script share module scope. A function defined in `_text_utils.py` is callable from any script that includes it. Variables like `PROJECT_FILE_PATH = "{PROJECT_FILE_PATH}"` declared in a helper are visible to the main script. **If a tool script uses a name, either it or one of its declared helpers must define it.** (See "Known pitfall" #1.)

## Substrate primitives that earn their keep

These exist for real reasons. Don't simplify them away without reading their CHANGELOG entry first.

| Primitive | File | Why it exists |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luke-harriman/Codesys-MCP](https://github.com/luke-harriman/Codesys-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

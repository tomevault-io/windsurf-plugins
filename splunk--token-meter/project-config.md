---
trigger: always_on
description: generates root `AGENTS.md` and `CLAUDE.md` discovery entry points from it; edit
---

# Coding-Agent Instructions

This is the maintained coding-agent instruction source. The project toolchain
generates root `AGENTS.md` and `CLAUDE.md` discovery entry points from it; edit
this file, then run `./scripts/setup-agent-tools`. The current engineering map
is [ARCHITECTURE.md](ARCHITECTURE.md).

AI-agent context for Token Meter, a local cross-platform dashboard and native companion for Claude, Codex, Cursor, OpenCode, and Kiro usage.

## Context

Token Meter reads local agent traces, calculates clearly labeled usage estimates, and serves a browser dashboard plus a native macOS menu-bar companion. The Python server is dependency-free and local-only. Mistakes can misstate cost, expose private trace data, or leave the installed runtime out of sync with the repository.

## Key Paths

| Path | Purpose |
|---|---|
| `meter.py` | Small executable/import compatibility facade for `token_meter.app` |
| `token_meter/app.py` | Composition, compatibility exports, settings, and application lifecycle |
| `token_meter/runtimes/` | Registered runtime discovery, parsing, revisions, and safe projections |
| `token_meter/platforms/` | Host paths, process/update policy, and trash behavior |
| `token_meter/domain/` | Runtime-neutral usage, timing, tools, insights, and aggregates |
| `token_meter/projections.py` | Explicit allowlisted public compatibility projections |
| `page.html` | Entire browser dashboard: markup, styles, routing, and JavaScript |
| `menubar/TokenMeterMenuBar.swift` | Native AppKit companion, preferences, notifications |
| `token_meter_mcp.py` | Bounded read-only MCP interface |
| `tests/test_meter.py` | Server, parser, UI-contract, installer, and Swift-source tests |
| `tests/test_mcp_server.py` | MCP contract and privacy tests |
| `runtime-manifest.txt` | Shared source-to-runtime packaging contract |
| `scripts/install` | Stage user runtime and install both macOS LaunchAgents |
| `scripts/install-windows.ps1` | Stage the same manifest and install Windows lifecycle/tray launchers |
| `README.md` | User installation and behavior documentation |
| `specs/ARCHITECTURE.md` | Canonical component boundaries, data flow, invariants, and extension budgets |
| `specs/CONTRIBUTING.md` | Human contribution policy and extension recipes |
| `specs/plans/active.md` | Ignored local execution state for multi-file work |
| `.agents/skills/` | Canonical project capability skills shared by agent hosts |
| `.agents/roles/` | Canonical specialist role boundaries and result contracts |
| `.agents/workflow/` | Routing, review gates, and structured handoff contracts |
| `agent-toolchain.lock.yaml` | Supported hosts, generated adapters, and pinned external skills |
| `scripts/setup-agent-tools` | Generate deterministic project-local host adapters |
| `scripts/check-agent-tools` | Read-only drift and external-skill validation |

## Commands

| Command | Purpose |
|---|---|
| `python3 -m unittest discover -s tests -v` | Run all unit and contract tests |
| `PYTHONPYCACHEPREFIX=/private/tmp/token-meter-pycache python3 -m py_compile meter.py token_meter_mcp.py $(find token_meter -type f -name '*.py' -print)` | Compile Python without polluting the repo |
| `node -e "const fs=require('fs');const h=fs.readFileSync('page.html','utf8');const m=h.match(/<script>([\\s\\S]*)<\\/script>/);new Function(m[1]);console.log('js ok')"` | Parse embedded dashboard JavaScript |
| `bash -n scripts/install scripts/install-linux scripts/install-launch-agent scripts/install-systemd-user scripts/run-menubar scripts/run-token-meter-mcp scripts/start-token-meter scripts/uninstall-launch-agent scripts/uninstall-systemd-user scripts/update scripts/update-linux` | Check shell syntax |
| `swiftc menubar/TokenMeterMenuBar.swift -o /private/tmp/token-meter-menubar` | Compile the native companion |
| `TOKEN_METER_MENUBAR_SMOKE=1 /private/tmp/token-meter-menubar` | Run deterministic native smoke output |
| `powershell -NoProfile -Command "[void] [scriptblock]::Create((Get-Content -Raw scripts/install-windows.ps1))"` | Parse a Windows script on a Windows host |
| `./scripts/install` | Stage and start the exact repository runtime |
| `curl -fsS http://127.0.0.1:8722/health` | Verify server/page readiness |
| `curl -fsS http://127.0.0.1:8722/menubar` | Verify compact native payload |
| `git diff --check` | Reject whitespace errors |

## Rules & Patterns

- Treat `README.md` and `specs/CONTRIBUTING.md` as human documentation; keep this file dense and agent-specific.
- Treat `specs/ARCHITECTURE.md` as the canonical engineering map. Link to it instead of copying a second component inventory.
- For multi-file or multi-milestone work, create or replace `specs/plans/active.md` before code edits.
- Keep `specs/plans/active.md` current with goal, decisions, progress, validation, and remaining work at every stopping point.
- `specs/plans/active.md` is ignored local execution state: never stage or commit it.
- Preserve unrelated worktree changes. Do not reset, checkout, or rewrite user changes.
- Keep `meter.py` and `token_meter_mcp.py` on the Python standard library.
- Keep the dashboard local-only; do not add hosted assets, analytics, or telemetry.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [splunk/token-meter](https://github.com/splunk/token-meter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

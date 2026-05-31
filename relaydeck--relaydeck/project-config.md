---
trigger: always_on
description: relaydeck is a **micro-agent orchestrator & workflow engine** —
---

# Notes for AI coding agents on this repo

relaydeck is a **micro-agent orchestrator & workflow engine** —
**web-primary**, plugin-extensible, with a fully scriptable CLI at parity.
One daemon per machine, web dashboard at `localhost:8765` (the primary operator
surface), a fleet of CLI-harness-wrapping agents per registered workspace,
durable agent-to-agent messaging, and a meta layer (`purpose`/`tags`) that lets
agents discover and delegate to each other.

**Tests:** `uv run pytest -q` (full suite green; `-m "not e2e"` in CI). Use
`uv run relaydeck …` in dev — a bare `relaydeck` may be a frozen tool copy that
ignores repo edits.

## Hard rules (don't violate without strong reason)

- **Everything is a plugin.** Harnesses, vault, metering, messaging, skills —
  all discovered at startup. No capability lives outside the plugin system.
  **All official plugins live in the root `plugins/` package**; `relaydeck/` is
  the engine + host contract. **The boundary is one-directional and enforced**:
  core NEVER statically imports `plugins`; plugins import only the public facades
  — `relaydeck.{sdk,harness,provider,vault,automation,testing}`. Strategy +
  governance: **Package separation** (below) and [CONTRIBUTING.md](CONTRIBUTING.md).
- **One primitive: the agent.** Every long-lived thing the daemon does is a
  `BaseAgent` subclass (`agents_base.py`). No parallel runtime surfaces.
- **YAML is the spec, SQLite mirrors it.** Agent defs live in
  `~/.relaydeck/agents/<id>.yaml` — source of truth for `id`/`name`/`type`/
  `workspace`/`config`/`auto_start`/`purpose`/`tags`/`system_prompt`/
  `inject_identity_preamble`. The DB mirrors a subset for fast queries. Never use
  the DB row as source of truth; `Orchestrator.start()` resyncs DB from YAML.
- **Workspace plugin source-of-truth is `agent.toml`, not `config.toml`.** The
  harness reads `workspaces/<ws>/agent.toml` at spawn; `load_workspace_registry()`
  prefers it, falling back to `config.toml` for legacy workspaces only.
- **The web dashboard is THE primary interface; the CLI is at parity, not
  ahead.** Anything doable from the CLI MUST be doable from the dashboard, both
  calling the same daemon HTTP API. **Never ship a web view that punts the
  mutation to the CLI** — no "use `relaydeck X`" empty-state where a form
  belongs. A new capability ships with its web affordance in the same change.
  The only legit CLI/TTY-only ops are inherently local-process (`daemon
  start/stop`, `serve`, `attach`, `view`, `workspace view`, `layout`).
  **Destructive/interrupting web actions must be responsible**: surface
  consequences + require confirmation (e.g. the daemon-restart confirm modal).
- **CLI ↔ daemon over HTTP, never via local orchestrator.** Live state (PTY
  injection, event subscribers, running instances) lives in the daemon. A CLI
  command that affects live state POSTs to the daemon, falling back to
  durable-enqueue when unreachable. Local-orchestrator calls are fine for DB
  reads + YAML edits, never for agent push.
- **Workspace plugins.** Workspaces declare enabled plugins in `agent.toml`
  (`plugins = ["messaging", "skills", ...]`); each harness checks the list and
  applies injections — no "modes". Only `workspace_scoped = True` plugins belong
  here, plus the harness-gate names in `workspace_plugins.py:HARNESS_GATES`.
  **Exception:** the GitHub poller starts when `workspaces/<ws>/github.yaml` has
  a valid `repo:` — not from `agent.toml`.
- **One source of truth for skills: `relaydeck/skills.py`.** Discovery, parsing,
  validation, hashing, materialization all live here; harnesses MUST use it so
  the Skills lens reports the *exact* set the model sees. Invalid skills (no
  `SKILL.md`, or frontmatter missing `name`/`description`) are skipped
  identically across harnesses. The **`skills` harness-gate** gates user-authored
  `workspaces/<ws>/skills/`; the bundled **`skills` plugin** is the inventory +
  management layer + the `[plugin.skills]` materializer.
- **Plugin-contributed skills go through `[plugin.skills]` + ownership sidecars
  — never hand-rolled.** A plugin declares `[plugin.skills]\n<name> =
  "SKILL.md"`; the `skills` plugin materializes into `runtime/skills/<name>/`
  with a `.relaydeck-skill.json` sidecar (`owner_plugin`, `source_hash`,
  `managed_by`) so plugins can't clobber each other and re-writes are
  hash-skipped. Optional duck-typed hooks: `skill_target_workspaces(all)` and
  `skill_content(name, source_text)`. Disabling `skills` stops re-sync but never
  strips materialized skills.
- **Vault values never leave the daemon.** `/api/vault/keys` returns names only.
  `${vault:NAME}` substitution happens at agent start; substituted config never
  round-trips to disk.
- **Migrations are additive.** `relaydeck/db.py:_migrate` runs on every
  `open_db`. New columns/tables only, guarded by try/except. Never rename, drop,
  or alter. Indexes on new columns live in `_migrate`, not `SCHEMA`.
- **Streaming is SSE + WebSocket.** SSE for events, WebSocket for harness PTY
  output. Never persist PTY bytes as events.
- **The dashboard is real-time by default — no per-view polling.** **Server**:
  one SSE feed (`_bus`) backs `/api/events`; `BaseAgent.log_event` publishes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [relaydeck/relaydeck](https://github.com/relaydeck/relaydeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->

---
trigger: always_on
description: **README.md** is for humans (install, features, links). **This file** is for coding agents editing the plugin or debugging hook behavior. Keep changes aligned with both; do not duplicate the full README here.
---

# oh-my-grok — agent guide

**README.md** is for humans (install, features, links). **This file** is for coding agents editing the plugin or debugging hook behavior. Keep changes aligned with both; do not duplicate the full README here.

Inspired by [oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent). Upstream handoff/Ralph/boulder behavior is ported and adapted for **Grok Composer** + `grok plugin`.

---

## What this repository is

| Is | Is not |
|----|--------|
| A **Grok plugin** (`plugin.json`, `hooks/hooks.json`, bundled skills/rules) | A standalone CLI or application users run from this repo |
| Go hook binary (`bin/omg-hook-*`) + thin `hooks/run-hook.sh` | User application code |
| Install target: `grok plugin install github:mihazs/oh-my-grok --trust` | Global copies under `~/.grok/hooks/` (deprecated; plugin-only) |

After install, Grok loads hooks from `GROK_PLUGIN_ROOT` (installed copy under `~/.grok/installed-plugins/oh-my-grok-*`, often symlinked to a local clone).

---

## Architecture (30-second map)

```
plugin.json
hooks/hooks.json          → SessionStart, UserPromptSubmit, Pre/PostToolUse, Stop, SessionEnd
hooks/run-hook.sh         → dispatches to bin/omg-hook-<os>-<arch>
cmd/omg-hook + internal/  → all hook logic (see docs/superpowers/plans/2026-06-02-go-hooks-migration.md)
  cmd/user_prompt.go      → single merged additionalContext (do not split into multiple JSON hooks)
  cmd/stop.go             → ralph → boulder → todo → lsp → plan.md (first block wins)
  internal/skillgate/     → catalog, PreTool gate, reminders
  internal/intentgate/    → keyword modes
  internal/prometheus/    → /plan, plan-mode PreTool guard
  internal/hashline/      → read cache, LINE#ID PreTool guard
  internal/lsp/           → diagnostics stash, post-tool + Stop
  internal/ralph/         → /ralph-loop, /ulw-loop, /cancel-ralph
  internal/boulder/       → boulder + todo continuation + todo enforcer state
skills/*/SKILL.md         → user-invocable workflows (discovered by grok inspect)
rules/*.md                → injected on every UserPromptSubmit (with workspace AGENTS.md)
```

**superpowers** skills are **bundled** at `vendor/superpowers/skills/` (`plugin.json` `skills` paths). Do not register duplicate oh-my-grok hooks globally. Refresh vendor: `task vendor:superpowers`.

---

## Two state namespaces (do not confuse)

| Location | Owner | Examples |
|----------|--------|----------|
| **`~/.grok/`** | Grok harness | `installed-plugins/`, `state/skill-gate/`, `state/hashline/`, `state/lsp-diagnostics/`, `state/todo-enforcer/`, `state/stop-continuation/`, `sessions/` |
| **`.omg/`** (per workspace) | oh-my-grok runtime | `boulder.json`, `plans/`, `todos/`, `ralph-loop.local.md`, `handoffs/` |

Analogous to omo’s **`.omo/`** in OpenCode workspaces. Never store plugin source or session catalogs under `.omg/`.

---

## Bundled skills & slash commands

| Skill | Command | Hook involvement |
|-------|---------|------------------|
| `agent-skill-gate` | (meta; Read before mutating) | `session-start`, `user-prompt`, `pre-tool-use`, `post-tool-read` |
| `ralph-loop` | `/ralph-loop "task"` | `user-prompt`, `stop` |
| `ulw-loop` | `/ulw-loop "task"` | same + Oracle verification pending |
| `cancel-ralph` | `/cancel-ralph` | clears `.omg/ralph-loop.local.md` |
| `handoff` | `/handoff` | `user-prompt` injects PHASE 0–4 instructions |
| `prometheus-plan` | `/plan`, `/prometheus` | `user-prompt` + `pre-tool-use` |
| `hashline-edit` | (workflow) | `hashline` package, `post-tool-read` |
| `ast-grep` | MCP tools | `.mcp.json` + `vendor/ast-grep-mcp` |
| `lsp` | MCP + hook stash | `post-tool-lsp`, Stop step 4 (optional `node`) |

User-facing pause/resume: `/stop-continuation`, `/resume-continuation` (see `rules/12-todo-boulder.md`).

Full event map and stop priority: **`hooks/README.md`** (read when touching Stop or UserPromptSubmit).

---

## Where to look (progressive disclosure)

| Task | Read first |
|------|------------|
| Install / publish / repo URL | `README.md`, `docs/installation.md` |
| Hook events, stop chain, `.omg/` layout | `hooks/README.md` |
| Skill-gate behavior | `skills/agent-skill-gate/SKILL.md`, `rules/00-agent-skill-gate.md` |
| Ralph / ultrawork | `skills/ralph-loop/SKILL.md`, `skills/ulw-loop/SKILL.md`, `rules/10-ralph-loop.md` |
| Boulder + todos | `rules/12-todo-boulder.md`, `internal/boulder/` |
| Handoff format | `skills/handoff/SKILL.md`, `rules/11-handoff.md` |
| IntentGate / Prometheus / hashline / LSP | `hooks/lib/{intent-gate,prometheus,hashline,lsp}.sh`, `docs/configuration.md` |
| ast-grep MCP build | `scripts/build-mcp-runtimes.sh`, `vendor/ast-grep-mcp/` |
| Remove stale global install | `scripts/remove-global-overlays.sh` |

Do not paste entire skill bodies into this file. Load the path from `grok inspect` when implementing.

---

## Development workflow

1. Clone repo; set `export GROK_PLUGIN_ROOT="$(pwd)"` for local hook tests.
2. Edit `hooks/`, `skills/`, or `rules/` — see decision table below.
3. Run smoke tests (required before claiming done):

```bash
cd oh-my-grok
grok plugin validate .

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mihazs/oh-my-grok](https://github.com/mihazs/oh-my-grok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->

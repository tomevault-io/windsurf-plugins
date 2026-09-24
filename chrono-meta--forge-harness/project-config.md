---
trigger: always_on
description: > **Always-loaded layer.** Keep agent selection and rules that must govern every runtime here.
---

# AGENTS.md — forge-harness Runtime Entry Point

> **Always-loaded layer.** Keep agent selection and rules that must govern every runtime here.
> Load execution examples, compatibility history, and conditional procedures only through the
> imperative pointers below.

## Relationship to CLAUDE.md

| File | Scope | Audience |
|---|---|---|
| `CLAUDE.md` | Session rules, protocols, orchestration flow | Claude Code |
| `AGENTS.md` | Portable runtime rules, agent roles, dispatch boundaries | AI runtimes + humans |

`CLAUDE.md` governs Claude-native automation. This file is the portable entry point for Codex and
other non-Claude runtimes, which do not auto-load `.claude/rules/*.md`.

> **Whole map (any runtime, read first if you are new here)**: `docs/map/FH_MAP.md` — what FH is, how it is
> implemented (every diagram node is a real path, re-checked by `scripts/test_fh_map_paths_lanes.sh`), why it is
> trustworthy (gates · lanes · grade sources, with the client-hook caveat stated), and what is operator-local.
> Interactive diagrams: https://chrono-meta.github.io/forge-harness/

## Agent Registry

forge-harness ships 8 tracked agents. The user-mastery spectrum (`beginner` · `main-player` ·
`expert`) plus `challenger` supplies multi-persona review; the remaining agents serve harness
operations or steel-quench.

| Agent | File | Role | Invoked by |
|---|---|---|---|
| `beginner` | `plugins/fh-meta/agents/beginner.md` | First-contact cold read; finds onboarding friction | `sim-conductor` Area A, `marketplace-gate`, `install-wizard`, direct |
| `main-player` | `plugins/fh-meta/agents/main-player.md` | Engaged-user view; scopes Light/Midcore/Heavy usage | `sim-conductor` Area A/D-code, direct |
| `expert` | `plugins/fh-meta/agents/expert.md` | Web-grounded domain accuracy and current practice | `sim-conductor` Area E/D, paper review, direct |
| `challenger` | `plugins/fh-meta/agents/challenger.md` | Evidence-cited adversarial evaluation | `steel-quench`, `harvest-loop`, `sim-conductor`, direct |
| `fact-checker` | `plugins/fh-meta/agents/fact-checker.md` | Pre-recommendation duplicate and stale-fact search | Before new asset creation or recommendation |
| `hub-persona-auditor` | `plugins/fh-meta/agents/hub-persona-auditor.md` | External-facing pre-publication persona audit | `harness-pr-reviewer`, `sim-conductor`, direct |
| `quench-challenger` | `plugins/fh-commons/agents/quench-challenger.md` | Steel-quench attack plus concrete fix direction | `steel-quench` Wave 1, `install-doctor`, `marketplace-gate` |
| `persona-innovator` | `plugins/fh-meta/agents/persona-innovator.md` | Naming gaps, frame proposals, frontier signals | `sim-conductor` Area A, `harvest-loop`, direct |

Machine-readable mirror: `.claude/registry/agent_cards.json`.

> **Agent frontmatter must be valid YAML — and this bites non-Claude runtimes hardest.** Claude Code's
> loader is lenient (it accepted an unquoted `description:` containing `": "`); a strict YAML parser
> does not, and then **every key below the bad line is silently dropped** — including `tools:` and any
> `model:` floor. Measured 2026-08-11: one agent's multi-line unquoted `description` (with `user:` /
> `assistant:` lines inside it) invalidated its declared `tools: Read, Grep, Glob` and its `model: opus`
> floor; the agent ran with all tools and no pin. Keep `description:` to **one quoted line** and put
> examples in the body. `bash scripts/validate_yaml.sh` now covers `plugins/*/agents/*.md` as well as
> skills, and reports a zero-file scan as an instrument error rather than a pass.

### Tool restrictions

| Agent | Allowed tools |
|---|---|
| `challenger` | Read, Grep, Glob, WebSearch, WebFetch |
| `fact-checker` | Read, Grep, Glob |
| `hub-persona-auditor` | Read, Grep, Glob |
| `quench-challenger` | Read, Grep, Glob |
| `persona-innovator` | Read, Grep, Glob, WebSearch, WebFetch |
| `beginner` | Read |
| `main-player` | Read, Grep, Glob |
| `expert` | Read, WebSearch, WebFetch |

The table is the whole roster — all eight agents above appear here. *Three were missing until
2026-08-11; the omission read as "unrestricted" to anyone checking this page, which is the wrong
default for a table whose subject is restriction.* Cross-check with the files themselves rather
than trusting either side alone: the same audit found `challenger` documented here with a tool set
its file never declared at all.

## Runtime Boundaries

- **Two layers:** `tracks/`, `knowledge/`, and skill methodology are model-agnostic. Plugin agents,
  hooks, slash commands, and `.claude/rules/` automation are Claude-native.
- **Output residency:** reusable methodology and polished public guidance belong in `knowledge/`,
  `plugins/`, or `docs/`. Raw signals, operator observations, handoffs, audit logs, and private
  reasoning are private-first; do not infer that colocated directories share a repository.
- **Runtime authority:** a non-Claude sidecar returns evidence candidates, never the terminal verdict.
  The governor must source-close each finding against a local file hit, literal source span, or
  passing check. Governor agreement alone is not an anchor.
- **Sidecar write boundary:** a sidecar audits; it does not write to the target tree. Return

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrono-meta/forge-harness](https://github.com/chrono-meta/forge-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

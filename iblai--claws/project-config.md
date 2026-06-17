---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A library of drop-in [OpenClaw](https://github.com/iblai/iblai-claw-setup) / [NemoClaw](https://github.com/NVIDIA/NemoClaw) agent configurations, one per [ibl.ai](https://ibl.ai) solution segment. The 7 segments are `higher-education`, `k-12`, `enterprise`, `government`, `legal`, `financial-services`, and `medical-healthcare`. Each segment is a complete multi-agent system — a parent orchestrator plus specialist subagents — that a NemoClaw host imports by copying it into `/sandbox/.openclaw/`.

Each segment is published as its own repository (`github.com/iblai/<segment>-agents`). This `claws` repo (`github.com/iblai/claws`) is the umbrella: it aggregates the 7 segment repos as **git submodules** and carries only `README.md`, `CLAUDE.md`, and `.gitmodules` at its own root. Clone it with `git clone --recurse-submodules`, or run `git submodule update --init` after a plain clone, to populate the `<segment>/` directories.

This repository contains **only what the OpenClaw/NemoClaw runtime actually reads**. Configuration-only: no application code, no build system, no tests.

## Structure

Each segment lives at `<segment>/` in the repo root **as a git submodule** pointing at `github.com/iblai/<segment>-agents`. Within each submodule:

- `openclaw.json` -- gateway configuration; every agent declared in `agents.list[]`
- `.config-hash` -- sha256 of `openclaw.json` (recompute after any edit)
- `.env.example` -- template for the OpenClaw daemon env file `~/.openclaw/.env` (deployer copies it there and fills in real credentials)
- `workspace/` -- shared writable workspace (`/sandbox/.openclaw/workspace/` at runtime)
- `skills/<tool>/SKILL.md` -- one skill directory per major third-party tool
- `agents/<agent-id>/agent/` -- per-agent workspace files (below)

## What the runtime reads

The OpenClaw runtime loads, per agent, **only** these workspace files. The repo contains exactly these and nothing else:

| File | Purpose |
|------|---------|
| `IDENTITY.md` | Agent persona — name, role, vibe |
| `SOUL.md` | Behavioral guidelines |
| `USER.md` | User/environment context — optional, as-needed |
| `TOOLS.md` | Tool/integration reference notes **and** data sources (`## Data Sources` section) |
| `AGENTS.md` | Multi-agent routing table — parent agent only |
| `HEARTBEAT.md` | Periodic awareness checklist — optional, for proactive agents |
| `BOOTSTRAP.md` | One-time first-run setup — optional, as-needed |
| `MEMORY.md` | Seed long-term facts — optional, for rules-heavy agents |
| `auth-profiles.json` | Per-agent LLM provider credentials (sample placeholders) |

Everything else — model, config, sandbox/security policy, skills wiring — lives in `openclaw.json`, **not** in per-agent files. There is no `MODEL.md`, `CONFIG.json`, `SECURITY.md`, or `DATA.md`: the runtime never read them.

## `openclaw.json`

- `agents.defaults` — shared `model`, `subagents` limits, `skills` (array of skill names), and `sandbox` block (`mode`, `backend`, `scope`, `workspaceAccess`).
- `agents.list[]` — one object per agent: `id`, `name`, `agentDir`, `model`, `identity`, `tools`, optional `heartbeat` and `session`. The parent carries `default: true` and `subagents.allowAgents`.
- Security/sandbox policy is the `sandbox` block — OpenClaw has no granular per-domain egress allowlist; host-level network filtering is a deployment concern.
- Never set the blocked config paths: `gateway.auth`, `gateway.controlUi.dangerouslyDisableDeviceAuth`, `tools.exec.host`, `sandbox.mode`, `hooks.allowUnsafeExternalContent`.

## Skills

A skill is a directory `skills/<tool>/SKILL.md` with YAML frontmatter:

- `name` -- skill identifier (referenced by `agents.defaults.skills`)
- `description` -- one line, exposed to the model
- `metadata` -- a **single-line** JSON object declaring required env vars, e.g. `metadata: {"openclaw":{"requires":{"env":["CANVAS_API_TOKEN"]}},"primaryEnv":"CANVAS_API_TOKEN"}`

Skill credentials resolve from environment variables (the OpenClaw daemon loads `~/.openclaw/.env`). `.env.example` documents every variable across all skills.

## Agent Model

- Each segment has ONE parent agent (`default: true`, id `<segment>-assistant`) and N specialist subagents.
- The parent delegates via the `sessions_spawn` tool; allowed children are in its `subagents.allowAgents`, routing is documented in `AGENTS.md`.
- Subagent ids are kebab-case with an `-agent` suffix. All agents default to `anthropic/claude-sonnet-4-5-20250929`.

## Installing a segment

A segment installs onto an OpenClaw/NemoClaw host (set up per [iblai-claw-setup](https://github.com/iblai/iblai-claw-setup)) by cloning the segment's own repo (`github.com/iblai/<segment>-agents`) — or the umbrella `claws` repo with `--recurse-submodules` — and copying the config files into the config root (`/sandbox/.openclaw/` for NemoClaw, `~/.openclaw/` for standalone OpenClaw), excluding `.git/`, then filling credentials, recomputing `.config-hash`, and reloading the gateway. Installing one segment installs its parent plus every subagent at once.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iblai/claws](https://github.com/iblai/claws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

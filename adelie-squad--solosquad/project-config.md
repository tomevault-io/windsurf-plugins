---
trigger: always_on
description: > **Canonical workspace guide.** Every AI tool working on this codebase
---

# SoloSquad — AGENTS.md

> **Canonical workspace guide.** Every AI tool working on this codebase
> (Claude Code, Codex, Aider, Cursor, OpenHands, etc.) reads this file.
> **Human-edited only — no AI agent may modify this file.** (CLAUDE.md
> at the repo root is a thin redirect kept for backward compatibility.)

A 24/7 AI assistant system for solo founders. Powered by Claude Code +
messenger bot (Discord/Slack) + automated crons + team-based agents.

## Core Philosophy

```
Output ≠ Goal. Output = Means to achieve the goal.
```

## Tech Stack

TypeScript + Node.js. Distributed as npm package.

```bash
npm install -g solosquad
solosquad init          # Setup wizard
solosquad bot           # Start messenger bot
solosquad cron start    # Start automated scheduler
solosquad status        # Dashboard
solosquad update        # Self-update (OpenClaw-style)
solosquad doctor        # Environment diagnostics
solosquad cron run      # Manual cron execution
solosquad migrate       # Upgrade workspace layout across versions
solosquad add org       # Add another organization to workspace
solosquad add repo      # Clone (URL) or register (local path) a repository
solosquad sync          # Sync org/repositories/ with .org.yaml
solosquad chief status / reset / compact          # v0.3.0 (PM→Chief, v1.1) — Chief session ops
solosquad workflow list / show / focus         # v0.3.0 — workflow inspect
solosquad rollback --workflow <id>             # v0.3.0 — snapshot revert
solosquad goal new / list / show / run / status / stop / verify   # v0.4
```

## Project Structure

```
package.json                        → npm package config
tsconfig.json                       → TypeScript config
bin/solosquad.ts                    → CLI entry point
src/
  cli/                              → CLI commands (init, bot, cron, status, update, doctor,
                                       agent, workflow, goal, memory, readiness)
  bot/                              → Chief runner, claude-process factory, session-store,
                                       events, agents-builder, workflow-reconciler,
                                       slash-commands, git-snapshot, workspace-meta,
                                       spawn-assembler (v0.6 8-layer JIT),
                                       agent-budget (v0.6 — author-budget 일반화), mention-parser (v1.0.1 — @<slug> multi-repo intent), repo-registry (v1.0.1)
  messenger/                        → Platform adapters (Discord, Slack)
  scheduler/                        → Cron-based cron execution + memory,
                                       trajectory-extractor (v0.6 §3),
                                       freq-keyword-miner (v0.6 §3.4),
                                       v06-stats-extract (v0.6 §2.5)
  memory/                           → FTS5 archive schema + rotate (v0.6 §4)
  util/                             → Config, paths, logger, cost
  engine/                           → v0.4 autonomous engine + stop-hook-adapter (v0.6 §5b)
  migrations/scripts/               → 0.5.0-to-0.6.0.ts (dry-run + apply)
agents/                             → Agent definitions (v1.1 flat layout — canonical bundle)
  main/{agent}/SKILL.md             → 5 main bots: chief, pm, engineer, designer, marketer
  specialists/{agent}/SKILL.md      → 20 specialists, flat (no team folder nesting)
teams/{team}/                       → 4 teams (product/engineering/design/marketing)
  composition.yaml                  → Team membership (= data, not folders) + main supervisor
  KNOWLEDGE.md                      → Team(=domain) shared knowledge (v0.6 §2.1)
  OKR.md                            → Quarterly OKR (Chief-authored)
skills/{skill}/SKILL.md             → Reusable skills (PM Tier-1/2 + Chief-invoked)
crons/{cron}.md                     → Bundled cron prompts (editable): morning/evening brief,
                                       pm-compaction, leading-indicator, *-housekeeping, *-rotate
                                       (top-level since v1.3.3; was assets/routines/)
knowledge/                          → Bundled workspace knowledge starter (v0.6 §2.3; top-level since v1.1)
user/                               → Bundled owner profile: profile.md, voice.md, preferences.md
                                       (replaced assets/core/, removed v1.3.1 §9)
assets/                             → Remaining bundled defaults staged into .solosquad/ on init
  docker/                           → Dockerfile + docker-compose.yml (v1.2.10)
  .env.example                      → Sample environment file
```

> **Agent layout (v1.1):** specialists live flat at `agents/specialists/{name}/`;
> team membership is *data* in `teams/{team}/composition.yaml`, not folder nesting.
> The old team-nested `assets/agents/{team}/` roster was removed in v1.3.1.
> Path resolution (`util/paths.ts getAgentsDir`): `.solosquad/agents/` →
> top-level `agents/` → bundle.

## 3-Layer Context (v0.6 topology)

> **Memory layering (v1.4.0 S-3 formalization).** The stores below map to a
> working / episodic / long-term model: working = current session jsonl +
> chief events; episodic = `memory/cron-logs/*.jsonl` (7-day hot) + git
> snapshots; long-term = `archive.sqlite` FTS5 + `knowledge/` + `memory/pm-skills/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Adelie-Squad/solosquad](https://github.com/Adelie-Squad/solosquad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

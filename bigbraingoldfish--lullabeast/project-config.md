---
trigger: always_on
description: This file is the complete orientation for a contributor or Claude Code session working in this repo. Read it before touching anything else. All facts are drawn directly from source files; check the cited paths if anything is surprising.
---

# CLAUDE.md — Lullabeast Repository Guide

This file is the complete orientation for a contributor or Claude Code session working in this repo. Read it before touching anything else. All facts are drawn directly from source files; check the cited paths if anything is surprising.

> **New here as a human contributor?** Start with **[README.md](README.md)** (what Lullabeast is + quickstart) and **[SETUP.md](SETUP.md)** (install and run). This file is the exhaustive *technical* orientation — long by design; skim to the section you need rather than reading top to bottom.

---

## What This Repo Is

Lullabeast is an autonomous multi-agent software development pipeline that orchestrates four LLM agents (planner, executor, reviewer, escalation) through a deterministic gate-based loop to iteratively build software from a roadmap. It depends on OpenClaw as external infrastructure (webhook server, agent session management, workspace directories). Lullabeast does not embed OpenClaw — it calls it.

This repo contains:
- The pipeline orchestration code (previously lived in `~/.openclaw/`, now migrated here)
- The UI dashboard (FastAPI backend + single-file React frontend)
- Agent identity documents deployed into OpenClaw workspaces at install time
- The full skill library (27 SKILL.md files)
- Pipeline tests and UI tests in separate directories

---

## Repository Structure

```
autodev-ui/
├── autodev/
│   ├── pipeline/
│   │   ├── orchestrator.py        # Main pipeline loop, state machine, all agent coordination
│   │   ├── sentinel_poller.py     # Sentinel + idle detection for agent completion
│   │   ├── skill_manager.py       # Per-phase skill injection into agent workspaces
│   │   ├── webhook_client.py      # OpenClaw webhook invocation
│   │   ├── heartbeat_cron.py      # Crash recovery watchdog (run by system cron)
│   │   ├── session_cleanup.py     # Session TTL pruning cron
│   │   └── gate_scripts/
│   │       ├── utils.py            # Shared gate utilities, error codes, atomic writes
│   │       ├── phase_resolver.py   # Roadmap parser + phase identification (formerly roadmap_parser.py)
│   │       ├── repo_init_check.py  # Git repo readiness gate
│   │       ├── planner_gate.py     # Evaluates planner output
│   │       ├── executor_gate.py    # Evaluates executor output, file manifest, unaccounted deletions
│   │       └── reviewer_gate.py    # Evaluates reviewer output, 3-pass attribution routing
│   ├── skill-library/
│   │   ├── {discipline}/{role}/SKILL.md   # 27 files: 7 phase-injectable disciplines × 3 roles (21)
│   │   │                                   # + completion/reviewer, prd-creator/readiness-reviewer,
│   │   │                                   # and 4 roadmap-converter skills
│   │   └── legacy/                        # Pre-discipline-library skills (historical)
│   ├── agents/
│   │   ├── {planner,executor,reviewer,escalation,prd-creator}/
│   │   │   ├── IDENTITY.md, SOUL.md, TOOLS.md, AGENTS.md, USER.md
│   │   │   └── HEARTBEAT.md  # planner, executor, reviewer only
│   ├── config/
│   │   ├── skill_mapping.yaml      # Maps roadmap subsystem prefixes → skill disciplines
│   │   └── mcp.json                # MCP server configuration
│   ├── tests/                      # Pipeline-level tests (orchestration, sentinel, skills)
│   └── docs/
│       ├── PIPELINE-SPEC.md        # Architecture spec — single source of truth (~1,900 lines)
│       ├── PIPELINE-CONSTRAINTS.md # Known issues, hardware limits, model bugs (~520 lines)
│       └── ASSUMPTIONS.md          # Live spec divergences and resolved ambiguities
├── ui/
│   ├── server.py                   # FastAPI server (~10,700 lines) — all API endpoints
│   ├── roadmap_parser.py           # Display roadmap parser (all phases → list)
│   ├── index.html                  # Single-file React frontend (React/Babel/Tailwind vendored in ui/static/, in-browser transpile, no build step)
│   ├── config.example.json         # Template for local ui/config.json (committed)
│   ├── config.json                 # Local overrides — gitignored; copy from config.example.json
│   ├── requirements.txt            # fastapi, uvicorn, python-multipart, aiohttp
│   ├── autodev-ui.service          # systemd unit file (Linux / WSL2)
│   └── com.autodev.ui.plist        # macOS LaunchAgent — mirrors the systemd unit
├── tests/                          # UI server tests (~50 pytest files)
├── install.sh                      # Deployment script (14 steps, see SETUP.md)
├── SETUP.md                        # Human-facing setup guide
└── .env                            # Local path config (gitignored, written by install.sh)
```

### Architectural note: intentional single-file design

`ui/server.py` is deliberately a single large file (~10,700 lines). The UI was built TDD across 23 phases; keeping all FastAPI routes in one file avoids cross-module import complexity in a single-process server. Do not split it into sub-modules without a deliberate refactoring decision.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bigbraingoldfish/lullabeast](https://github.com/bigbraingoldfish/lullabeast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->

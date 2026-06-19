---
trigger: always_on
description: This file provides guidance to Claude Code when working in this monorepo.
---

# CLAUDE.md

This file provides guidance to Claude Code when working in this monorepo.

## Project Overview

Agentic Coding Reference: a living reference for agentic coding principles, demonstrated through production-ready agent configurations in Go and Java Spring Boot.

This is a **documentation and reference** project, not an application. The primary output is guidance, patterns, and working agent configurations that others can learn from and adapt.

## Repository Structure

```
.
├── docs/                          # Cross-cutting principles and architecture
│   ├── specialist-agent-workflow.md
│   ├── harness-project-api.md
│   ├── agentic-harness.md
│   ├── ddd-principles.md
│   └── adr/                       # Decision log: why the harness evolved
├── harness/                       # Single canonical harness source — samples materialize from here
│   ├── core/                      # Runtime shared by every stack
│   ├── stacks/<stack>/            # Stack-specific runtime (go, java-spring-boot, generic)
│   ├── init/                      # Skeletons for project-owned files (not runtime)
│   ├── marketplace/               # Producer-side assets for the marketplace channel (hooks.json, setup.sh, setup-skill.md)
│   └── *.sh                       # materialize / init / bootstrap / package-marketplace / check-sync
├── tools/                         # Repo-level tooling shared across samples
│   └── harness-stats/             # Statusline + cache-report scripts (user-level install)
├── samples/                       # Materialized instances of the harness (copy channel)
│   ├── go/                        # Materialized Go instance
│   │   ├── CLAUDE.md              # Go-specific agent instructions (authoritative)
│   │   └── ...                    # docs/ + scripts/layout.toml + runtime all committed
│   └── java-spring-boot/          # Materialized Spring Boot instance
│       ├── CLAUDE.md              # Spring Boot-specific agent instructions (authoritative)
│       └── ...
├── .claude-plugin/                # Generated: marketplace.json (the reference IS a marketplace)
├── plugins/                       # Generated: per-tool plugins, rendered by package-marketplace.sh
└── README.md
```

## Self-Contained Implementations

The `samples/go/` and `samples/java-spring-boot/` directories are **self-contained projects** — each with its own committed `CLAUDE.md`, `docs/` briefs, `scripts/layout.toml`, and build toolchain. Their runtime (agents, skills, hooks, schemas, engines) is materialized from `/harness` on the copy channel and committed. When working inside either:

- Follow that project's `CLAUDE.md` — it is the authoritative source for build commands, conventions, and agent workflow.
- Do not apply Go conventions to Java or vice versa.
- The per-tool runtime (`.claude/agents/`, `.claude/skills/`, `.github/agents/`, `.opencode/agents/`, `.junie/agents/`) is materialized from `/harness` and committed on the copy channel. Edit the source in `/harness` and re-materialize; never hand-edit a sample's runtime copy.

## What to Do at the Root Level

At the monorepo root, work is limited to:

- **Editing `docs/`** — Cross-cutting principles, the specialist agent workflow guide, and any new documentation.
- **Editing `docs/adr/`** — The reference's decision log: why the harness evolved. Record harness-level architecture decisions here, not in the samples (samples ship no ADRs; a consumer's decision log is its own). Pair each milestone with a Project History entry in `README.md`.
- **Editing `harness/`** — The canonical harness source (`core/`, `stacks/<stack>/`, `init/`) the samples materialize from. Harness changes go here, then `harness/bootstrap.sh` re-materializes both samples; never hand-edit a sample's committed runtime. Keep `core/` stack-agnostic (no language-specific fact).
- **Regenerating the marketplace** — `.claude-plugin/` and `plugins/` are *generated* by `harness/package-marketplace.sh` from `/harness`. After a harness change, re-run it; never hand-edit the generated plugins (same rule as the samples). `check-sync.sh` fails if the committed marketplace drifts from source.
- **Editing `README.md`** — The project overview and navigation.
- **Editing this file** — Monorepo-level instructions.
- **Cross-project consistency** — Ensuring patterns described in `docs/` are reflected in both implementations.

The root carries the canonical harness *source* (`harness/`) but never *runs* the harness — no live pipeline, no `.scratch/` handoff ledger, no agent-teams continue-hook; those run in the samples that demonstrate it. The root authors and maintains the harness; the samples execute it.

## Root-Level Skills

| Skill | Purpose |
|-------|---------|
| `audit-harness` | Hold the reference to a high bar after a change: run the deterministic battery (`check-sync.sh`), then `audit-consistency`, then an adversarial review of the diff for regressions/lost-coverage/incoherence; end with one verdict |
| `release-prep` | Roll `/harness` out to every instance, then prove it green: `bootstrap.sh` (samples) + `package-marketplace.sh` (marketplace), then the full `check-sync.sh` battery. The propagate-and-verify step after a `/harness` edit; writes the tree, never commits or judges |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [woditschka/agentic-coding-reference](https://github.com/woditschka/agentic-coding-reference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->

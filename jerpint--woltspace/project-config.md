---
trigger: always_on
description: **Woltspace** is a platform for running autonomous AI agents called **wolts**. Each wolt is:
---

# Woltspace — Developer Guide

## What is this?

**Woltspace** is a platform for running autonomous AI agents called **wolts**. Each wolt is:
- An AI with a persistent identity, memory, and personality
- Living inside a Docker container (the "space") where it can build whatever it wants
- Connected to its human partner via messaging apps (Telegram, WhatsApp/Slack)

The human chats with the wolt via Telegram. The wolt can spin up Claude Code sessions to do real work — write code, build websites, generate content. The human can watch the work happen live via a split-view browser UI (terminal on left, viewport on right).

**This repo is the platform** — the Docker image, server, bot brain, and CLI that makes wolts run. Individual wolt data lives separately (under `~/.woltspace/wolts/{name}/`).

> **⚠️ Platform code is immutable from wolt sessions.** Wolts must NEVER edit files in `/workspace/woltspace/`. All wolt work happens inside their own directory (`/workspace/wolts/{name}/`). Apps go in `wolts/apps/`. If a wolt needs new platform functionality, the human files an issue — wolts don't patch the platform.

---

## Architecture at a Glance

```
Human (Telegram/Slack)
    ↓
Python bot (litellm → Claude/Haiku)
    ↓
Tool: claude_code → tmux session → Claude Code CLI
    ↓
Wolt builds things (site, sparks, artifacts)
    ↓
FastAPI server (port 7777) → split view in browser ← Human watches
    ↑
cloudflared tunnel (optional public URL, no account needed)
```

**Key design:** The woltspace repo is `git clone`'d into the Docker image at build time. The only host mount is `~/.woltspace/wolts/` — all wolt data. The container is disposable; the wolts directory is the backup.

---

## Lore — The Colony

**Woltspace** is the whole thing. Wolts live and work here.

**The lodge** — home base. Always-on, handles day-to-day chat.

**The dens** — where real work happens. Claude Code sessions, spawned per task, dissolved when done.

**The pond** — the visible surface (viewport). Wolts build in dens, surface work to the pond.

### The Animals

Every wolt has a fixed **creature type** (`"type"` in `wolt.json`). The type determines the model, tempo, and role. A wolt's type is permanent.

**Rodents** (default) — general-purpose builders. Unlimited per workspace. Skill levels within the family:
- **🦫 Beaver — Sonnet** — the builder. Industrious, shapes the environment, makes things that last. Den sessions run as beavers — this is where real work happens.
- **🦝 Raccoon — Opus** — the orchestrator. Clever, adaptive. Sees the whole pond from the water's edge — doesn't build everything itself but knows where everything fits.
- **🦦 Otter — Haiku** — the quick one. Fast, lightweight, cheap. Great for simple lookups, one-shot edits, quick scripts.

**Singleton creatures** — one active at a time per workspace. Each is its own wolt with its own identity.
- **🐶 Dog — Haiku** — the lodge companion. Loyal, constrained, always present on Telegram. Bot responses formatted as `🐶 name: text`. Created via `/telegram` skill.
- **🐺 Wolf — Sonnet** — the scheduler. Runs crons, reminders, recurring tasks. Manages `wolt/wolf.json`. Created via `/wolf` skill.

**Creature-type rules:**
- `wolt.json` → `"type"` field: `rodent` (default), `wolf`, `dog`, `spider`, `bear`, `panda`
- `woltspace.json` → `creatures.active_wolf` / `creatures.active_dog` tracks the active singleton
- Creating a new wolf/dog demotes the old one to rodent (with notification)
- Discovery: scan `/workspace/wolts/*/wolt/wolt.json` or use `lib/wolts.py`
- CLI: `create-creature-wolt <name> <type>` creates creature-wolts

```
woltspace
  lodge           — home base
    dogwolt 🐶    — always-on, lodge companion, entry point
    beaverwolt 🦫 — builder with memory and identity
    raccoonwolt 🦝 — orchestrator, spans the colony
    otterwolt 🦦  — quick tasks, fast and lightweight
    wolfwolt 🐺   — scheduler, fires crons and dispatches creatures
  dens            — temporary work sessions (where beaverwolts/raccoonwolts/otterwolts/wolfwolts build)
  pond            — the visible surface (viewport)
```

---

## Key Components

### `woltspace` (bash CLI)
The host-side CLI. A dumb pipe to Docker — no python3, no JSON parsing, no git required on host.

Commands:
- `woltspace init` — first-time setup (or reconnect existing wolts)
- `woltspace start` — start, restart, or resume container
- `woltspace stop` — stop and remove container
- `woltspace backup [tag] [--bundle]` — snapshot container + wolts (tag defaults to datetime, `--bundle` zips into one portable file)
- `woltspace rebuild` — rebuild image + restart
- `woltspace shell/chat/logs` — interact with running container

Flags:
- `--local` — build image from local repo (COPY) instead of git clone
- `--branch <name>` — build image from a specific branch (default: main)

Env vars:
- `WOLTS_DIR` — override wolts directory (default: `~/.woltspace/wolts`)
- `WOLTSPACE_LOCAL=true` — sticky equivalent of `--local` (for dev workflows)

The only mount is `$WOLTS_DIR:/workspace/wolts`. Everything else is baked into the image.

### Docker Image


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jerpint/woltspace](https://github.com/jerpint/woltspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->

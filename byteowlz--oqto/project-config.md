---
trigger: always_on
description: Oqto is a self-hosted platform for managing AI coding agents.
---

# Oqto - AI Agent Workspace Platform

Oqto is a self-hosted platform for managing AI coding agents.

**New to Oqto?** Start with the [SETUP.md](./SETUP.md) guide for installation and prerequisites.

---

## IMPORTANT

- Keep this document up to date. Whenever we change functionality or the architecture, we need to also update it in here so that subsequent sessions are always aware of the current status.
- Keep crate-level guidance up to date too: if behavior/boundaries change in `backend/crates/*`, update that crate's `AGENTS.md` in the same PR. Every crate under `backend/crates` must have a concise `AGENTS.md`.
- Don't keep legacy alive. This project is still in it's infancy and there is 0 need for any backward compatibility. Remove any dead or legacy code you encounter without breaking the current system. If you stumble upon parts of the system that can be deprecated, suggest how we could best do this
- Document your work: Use trx cli for epics, features, bugs etc. Use agntz memory for documenting learnings along the way. Future sessions have access to both.
- **No hacky fixes.** We want proper John Carmack solutions -- clean, minimal, and correct. Understand the root cause before writing a single line. If a fix feels like duct tape, stop and rethink. Every change should make the codebase better, not just silence the symptom.
- **Respect the architecture.** Actions go through Runners. History goes through hstry. Memory goes through mmry. Do not bypass established data flows. If you think you need a shortcut, you are missing something -- re-read the architecture section above and the canonical protocol docs.
- **"Let me just..." is ALWAYS wrong.** That phrase is the preamble to a hack. We do not "just" add a quick workaround, "just" hardcode a value, or "just" skip the proper path. Every solution must be designed to scale. If it would not survive 10x users or 10x sessions, it is the wrong approach.
- **Todo list discipline**: Your todo list is a real-time status bar the user watches. At the start of a task, create todos with `TodoWrite`. As you work, **always** update the list: set tasks to `in_progress` when you start them and `completed` when you finish them. Do not leave completed tasks as `pending`. Rewrite the full list after each significant step.
- Oqto is made up of many separate tools that we are building in parallel. If you encounter bugs or potential improvements, file trx in the respective repos (e.g. ../mmry etc)
- During development, use the agent-browser for end to end debugging. You can use wismut:dev to log in. The frontend is accessible under localhost:3000

## Debugging

Tmux is always available, use it to debug the logs of the running backend and frontend.

### agent-browser (headless browser testing)

Requires `DISPLAY=:0` prefix on all commands (X server runs on :0).

```bash
DISPLAY=:0 agent-browser open http://localhost:3000      # Open page
DISPLAY=:0 agent-browser snapshot -i                     # List interactive elements with refs
DISPLAY=:0 agent-browser fill @e1 "text"                 # Fill input by ref
DISPLAY=:0 agent-browser click @e2                       # Click by ref
DISPLAY=:0 agent-browser press Enter                     # Press key
DISPLAY=:0 agent-browser screenshot /tmp/shot.png        # Screenshot (view with Read tool)
DISPLAY=:0 agent-browser console                         # Browser console logs
DISPLAY=:0 agent-browser eval "JS expression"            # Run JS in page
DISPLAY=:0 agent-browser wait 3000                       # Wait ms
DISPLAY=:0 agent-browser scroll down 500                 # Scroll
DISPLAY=:0 agent-browser close                           # Close browser
```

Enable frontend debug logging: `DISPLAY=:0 agent-browser eval "localStorage.setItem('debug:pi-v2', '1')"`

---

## Architecture Overview

```
Frontend                          Backend                           Runner (per user)
   |                                 |                                    |
   |-- Single WebSocket ------------>|                                    |
   |   (multiplexed channels)        |                                    |
   |                                 |-- Unix/TCP socket ---------------->|
   |                                 |   (runner protocol)                |
   |                                 |                                    |
   |   {channel:"agent", ...}        |   Canonical Commands              |-- Agent Process A
   |   {channel:"files", ...}        |   Canonical Events                |-- Agent Process B
   |   {channel:"terminal", ...}     |                                   |-- hstry (gRPC)
```

### Core Components

| Component | Purpose |
|-----------|---------|
| **Frontend** | React/TypeScript app speaking the canonical protocol via multiplexed WebSocket |
| **Backend (oqto)** | Stateless relay: routes commands to runners, forwards events to frontend |
| **Runner (oqto-runner)** | Per-user daemon: owns agent processes, translates native events to canonical format |
| **hstry** | Chat history service (gRPC API, SQLite-backed). All reads/writes go through gRPC. |

### The Canonical Protocol


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [byteowlz/oqto](https://github.com/byteowlz/oqto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

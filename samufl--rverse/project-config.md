---
trigger: always_on
description: This file contains mandatory instructions for any AI coding agent working on this repository.
---

# AGENTS.md — RVRSE

This file contains mandatory instructions for any AI coding agent working on this repository.
Read it fully before touching any code.

---

## 1. Project Overview

RVRSE is a free, open-source audio plugin (VST3 / AU / CLAP) built with **iPlug2 and C++17**.
It generates a reverse-reverb riser automatically from any loaded hit sample and fires the hit
at a tempo-synced beat boundary. Full spec: [`RVRSE_BRIEF.md`](./RVRSE_BRIEF.md).

This is a **warmup project** for the larger OpenSampler initiative. Correctness and clean
architecture matter more than speed of delivery.

---

## 2. Task Tracking — Beads (`bd`)

This project uses **[Beads](https://github.com/steveyegge/beads)** (`bd`) for issue tracking.
Beads is a git-backed, agent-optimised issue tracker. All planning lives there, not in markdown
files or freeform TODO comments.

### Setup (once per machine)

```bash
# Install the bd CLI globally
curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash

# For VS Code + Copilot: install the MCP server
uv tool install beads-mcp
# Then add to .vscode/mcp.json:
# { "servers": { "beads": { "command": "beads-mcp" } } }

# Install git hooks in the repo (auto-syncs issues on commit/pull)
bd hooks install
```

### Mandatory Beads Workflow

Follow this pattern for every working session, without exception:

```
START OF SESSION
  bd prime              # Load context — read this output carefully
  bd ready --json       # Find unblocked tasks to work on

DURING WORK (for each task)
  bd update <id> --claim              # Claim the task before starting
  bd create "Sub-task" --type task    # Create child tasks as needed
  bd dep add <child-id> <parent-id>   # Link dependencies explicitly

END OF SESSION  ("land the plane")
  bd close <id> --reason "..." --json # Close completed tasks
  bd sync                             # Export + commit the issue database
  git pull --rebase
  git push                            # MANDATORY — do not stop before this
  git status                          # Must read "up to date with origin/main"
```

> **CRITICAL:** A session is NOT complete until `git push` succeeds and
> `git status` confirms you are up to date. Never say "ready to push" and stop.
> Push. Every. Time.

### Key Commands Reference

| Command | Purpose |
|---|---|
| `bd prime` | Load full workflow context — run at session start |
| `bd ready` | List tasks with no open blockers — your work queue |
| `bd create "Title" -t task -p 1` | Create a task (priority 0=highest, 4=lowest) |
| `bd update <id> --claim` | Atomically claim a task (sets you as assignee + in_progress) |
| `bd dep add <child> <parent>` | Mark that child is blocked by parent |
| `bd dep tree <id>` | Show full dependency tree for an issue |
| `bd show <id> --json` | View full task details |
| `bd close <id> --reason "Done"` | Mark a task complete |
| `bd sync` | Sync JSONL and commit |
| `bd stats` | Overall project progress |

### Commit Message Convention

Always include the Beads issue ID at the end of commit messages:

```
git commit -m "Add Schroeder reverb implementation (bd-a1b2)"
git commit -m "Wire stutter gate to MIDI CC (bd-c3d4)"
```

This lets `bd doctor` detect orphaned issues (committed but not closed).

---

## 3. Library Documentation — Context7

This project uses the **Context7 MCP server** to provide agents with up-to-date library
documentation. Before implementing any call to an external library or framework, **check
Context7 first** to verify the correct API.

### When to use Context7

- **iPlug2 API calls** — `IPlug`, `IGraphics`, `IControl`, `IParam`, `IMidiQueue`, etc.
- **dr_libs** — `dr_wav`, `dr_flac` header-only audio codecs.
- **C++ standard library** — when unsure about C++17 behaviour or edge cases.
- **CMake** — build system functions, `FetchContent`, target properties.
- **Any third-party dependency** added in the future.

### Rule

> **Do not guess library APIs.** If you are unsure about a function signature, parameter
> order, return type, or behaviour, use Context7 to look it up. Incorrect API usage in
> audio code causes subtle bugs that are painful to diagnose.

### Setup

Context7 is configured in `.vscode/mcp.json` (git-ignored — contains API key).
See [context7.com](https://context7.com) to obtain an API key.

---

## 4. Architecture Rules — Non-Negotiable

The codebase is split into two strictly separated layers. Violating this causes audio glitches
and subtle real-time bugs that are painful to debug.

### Offline Layer (`RvrseProcessor`)
Runs on a **background thread**. Never called from the audio thread.

- Sample loading
- Reverb application (`Reverb.h`)
- Buffer reversal
- Time-stretching (`Stretcher.h`)
- Writes the result into `final_riser[]`

### Real-Time Layer (`RvrseVoice`)
Runs in `ProcessBlock()` on the **audio thread**. Must be lock-free and allocation-free.

- Reads from `final_riser[]` (pre-computed, read-only during playback)
- Stutter gate (`Stutter.h`) — computed per-sample
- Fade envelope
- Hit playback at the calculated offset
- Responds to MIDI CC instantly

> **`Stutter.h` is real-time only.** It must never be called from the offline pipeline.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SamuFL/rverse](https://github.com/SamuFL/rverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

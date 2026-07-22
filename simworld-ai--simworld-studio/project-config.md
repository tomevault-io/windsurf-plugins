---
trigger: always_on
description: > This file is read by Codex, Claude Code, and all AI coding agents before doing any work.
---

# SimWorld Studio — Engineering Instructions

> This file is read by Codex, Claude Code, and all AI coding agents before doing any work.
> Keep it up to date. It is the single source of truth for how to build this project.

---

## Product

SimWorld Studio is a four-mode research platform for:

1. **Scene Generation** — Build and verify UE5 environments from text/image/edit prompts
2. **Task Generation** — Convert verified scenes into PointNav / ObjectNav / custom task sets
3. **Agent Training** — Run embodied agents, collect trajectories, evaluate metrics
4. **Co-evolution** — Adapt scene difficulty based on agent feedback in a closed loop

The UI follows a strict pipeline: Scene → Task → Training → Co-evolution.  
**Do not collapse all four modes into one screen.**

---

## Core Principle

> One mode = one job = one primary output.

| Mode | Primary Output |
|------|---------------|
| Scene Generation | Verified Scene (v3, v4…) |
| Task Generation | TaskSet (PointNav_500, ObjectNav_200…) |
| Agent Training | TrainingRun + trajectories + metrics |
| Co-evolution | CurriculumRun + adaptation decisions |

---

## Repository Structure

```
simworld_studio_workspace/
  web/
    src/
      App.jsx              ← single-file React app (current implementation)
      index.css            ← all styles, dark/light themes, CSS vars
    server/
      index.js             ← Node.js/Express API + WebSocket server
      mcp-server.js        ← MCP UE tool server
      agents.js            ← agent session management
      arena.js             ← arena battle logic
      skills.js            ← skill store
      scenes.js            ← scene persistence
    public/
      ue-player.html       ← Pixel Streaming iframe
  scenes/                  ← saved scene JSON files
  cirrus-config.json       ← Pixel Streaming signalling config

.codex/config.toml         ← Codex MCP configuration
.agents/skills/            ← Codex reusable skills
docs/
  product/                 ← product specification
  design/                  ← UI reference images
AGENTS.md                  ← this file
```

---

## Current Tech Stack

**Frontend** (`simworld_studio_workspace/web/src/`):
- React 18 (functional components, hooks)
- Vite (build + dev server)
- Single file: `App.jsx` (~9 000 lines, all components inline)
- `index.css` — CSS variables, dark/light themes
- No TypeScript yet — plain JSX
- Inter font, JetBrains Mono for code
- `react-markdown` + `remark-gfm` for markdown rendering

**Backend** (`simworld_studio_workspace/web/server/`):
- Node.js + Express
- Server-Sent Events (SSE) for real-time state push
- REST API at `/api/*`
- MCP server at port 55557 (UE5 tool bridge)
- JSON file persistence for skills/scenes

**Simulation**:
- Unreal Engine 5 (local, Pixel Streaming over WebSocket)
- MCP protocol bridge (`mcp-server.js`)
- Verifier: rule-based collision check + VLM semantic check
- Cirrus signalling server for Pixel Streaming

---

## Current Implementation State

### Already implemented (koe branch):

- ✅ Four studio modes: Scene Generation, Task Generation, Agent Training, Co-evolution
- ✅ `PipelineStepper` — numbered tab nav [1→2→3→4]
- ✅ `ArtifactChain` — shows Scene→Task→Training→Curriculum build state
- ✅ Mode-aware primary CTA button in navbar
- ✅ Mode-specific panel mapping:
  - Scene: Left=Intent+SimCoder, Right=Scene Inspector
  - Task: Left=Task Builder, Right=Task Inspector (placeholder)
  - Training: Left=Training Config, Right=Agent Monitor
  - Co-evolution: Left=Curriculum Builder, Right=Round Inspector
- ✅ Dark/Light themes via `data-theme` CSS variables
- ✅ Shared primitives: `Badge`, `Btn`, `ToggleBtn`, `TagChip`, `ModalOverlay`, `PageHeader`, `Eyebrow`, `Field`
- ✅ Library section (Skills + Tools + Arena as tabs)
- ✅ Results section (Gallery + Leaderboard as tabs)
- ✅ Real UE5 viewport via Pixel Streaming iframe
- ✅ Real MCP tool calls (spawn, delete, screenshot, etc.)
- ✅ Real SimCoder chat (Claude API)
- ✅ Real SSE state stream from server
- ✅ Embodied agent panel (live agent state, trajectory, stats)

### Still placeholder / needs real implementation:

- ⚠️ Task Generation (left+right panels are UI stubs — no backend)
- ⚠️ Training Config panel (stub — no Gym/PPO integration)
- ⚠️ Curriculum Builder (stub — no co-evolution orchestration)
- ⚠️ NavMesh overlay in Task Gen center view
- ⚠️ Agent first-person RGB in Training center view
- ⚠️ Co-evolution loop canvas (shows static diagram)
- ⚠️ Artifact chain (no persistence yet — ephemeral state)

---

## Frontend Routes (implemented as studioMode state)

```
studioMode === "scene"    → Scene Generation layout
studioMode === "task"     → Task Generation layout
studioMode === "training" → Agent Training layout
studioMode === "coevolve" → Co-evolution layout
```

```
topSection === "studio"   → 3-column studio layout
topSection === "library"  → Library page (Skills/Tools/Arena)
topSection === "results"  → Results page (Gallery/Leaderboard)
```

---

## Shared StudioShell

Every mode renders:
- `PipelineStepper` in navbar (4 pipeline tabs with arrows)
- Secondary nav: Studio | Library | Results
- Mode-aware primary CTA button
- `ArtifactChain` (shows built artifacts)
- Left column (mode-specific panel)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SimWorld-AI/SimWorld-Studio](https://github.com/SimWorld-AI/SimWorld-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

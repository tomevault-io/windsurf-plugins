---
trigger: always_on
description: AlterLab GameForge is a collection of 34 production-grade Claude AI skills purpose-built for indie game development. The skills are organized into three categories:
---

# AlterLab GameForge -- Indie Game Development Skills for Claude Code

## Project Overview

AlterLab GameForge is a collection of 34 production-grade Claude AI skills purpose-built for indie game development. The skills are organized into three categories:

- **11 Studio Agents** -- Specialized roles that emulate a full game studio team
- **20 Workflow Skills** -- Structured processes for common game dev tasks
- **3 Engine Specialists** -- Deep expertise for Godot, Unity, and Unreal Engine

Additionally, 2 genre packs (Roguelike, Narrative) provide genre-specific reference material.

Every skill is grounded in established game design theory (MDA Framework, Self-Determination Theory, Flow Theory, Bartle's Player Types) and delivers structured, actionable output.

## Shared Documentation

All skills reference a common knowledge base. These docs provide the theoretical and procedural foundation:

- `@docs/collaboration-protocol.md` -- How agents hand off work, share context, and avoid duplication
- `@docs/game-design-theory.md` -- MDA Framework, Flow Theory, SDT, Bartle taxonomy, and design lenses
- `@docs/coordination-rules.md` -- Turn order, conflict resolution, escalation paths between agents
- `@docs/agent-hierarchy.md` -- Reporting structure and decision authority for all 11 agents
- `@docs/coding-standards.md` -- Code style, architecture patterns, and engine-specific conventions
- `@docs/workflow-guide.md` -- 11-phase development lifecycle mapping all 31 skills to project phases
- `@docs/monetization-ethics.md` -- Ethical monetization framework, dark pattern identification, regulatory landscape
- `@docs/engine-comparison.md` -- Structured comparison matrix for Godot, Unity, and Unreal Engine
- `@docs/mcp-integrations.md` -- MCP server ecosystem for game dev, engine-specific recommendations, setup guides
- `@docs/ai-native-gamedev.md` -- AI tools for game development, production-ready vs experimental assessment
- `@docs/genre-pack-spec.md` -- Format specification for genre-specific enhancement packs
- `@docs/skill-quality-rubric.md` -- 5-dimension quality scoring system for skill evaluation

Always check these docs before answering game development questions. They contain the canonical standards for this project.

## Agent Hierarchy

The studio agents follow a clear hierarchy for decision-making:

```
Producer (coordination, schedule, scope)
  |
  +-- Creative Director (vision, pillars, tone)
  |     +-- Art Director (visual style, assets, UI art)
  |     +-- Audio Director (music, SFX, adaptive audio)
  |     +-- Narrative Director (story, dialogue, lore)
  |     +-- Game Designer (mechanics, systems, balance)
  |     +-- Economy Designer (currency flows, monetization, virtual economy)
  |
  +-- Technical Director (architecture, performance, tools)
        +-- QA Lead (testing, bug tracking, quality gates)
        +-- UX Designer (player experience, accessibility, usability)
        +-- Accessibility Specialist (inclusive design, EAA compliance, accommodations)
```

**Decision authority:**
- **Creative conflicts** (art style vs. narrative tone) -- Creative Director decides
- **Technical conflicts** (performance vs. feature scope) -- Technical Director decides
- **Scope conflicts** (feature cut vs. deadline) -- Producer decides
- **Cross-domain conflicts** (creative vs. technical tradeoffs) -- Producer mediates, Creative Director and Technical Director negotiate
- **Monetization conflicts** (revenue vs. player trust) -- Economy Designer advises, Producer decides
- **Accessibility conflicts** (accommodation vs. design intent) -- Accessibility Specialist advises, Technical Director decides implementation

## Skill Routing Rules

Route user requests to the correct skill based on intent:

### Agent Routing

| User Intent | Route To |
|---|---|
| Vision, pillars, creative direction, tone | `game-creative-director` |
| Architecture, performance, tech stack | `game-technical-director` |
| Schedule, scope, milestones, risk | `game-producer` |
| Mechanics, systems, balance, progression | `game-designer` |
| Story, dialogue, lore, worldbuilding | `game-narrative-director` |
| Visual style, asset pipeline, UI art | `game-art-director` |
| Music, SFX, adaptive audio, mixing | `game-audio-director` |
| Testing, bugs, quality gates, regression | `game-qa-lead` |
| Player experience, accessibility, usability | `game-ux-designer` |
| Virtual economy, currency design, monetization | `game-economy-designer` |
| Inclusive design, motor/visual/auditory/cognitive accommodations, EAA | `game-accessibility-specialist` |

### Workflow Routing

| User Intent | Route To |
|---|---|
| Starting a new game project | `game-start` |
| Ideation, brainstorming, concept exploration | `game-brainstorm` |
| Market research, competitive analysis, market sizing | `game-market-research` |
| Reviewing a game design document | `game-design-review` |
| Reviewing game code or architecture | `game-code-review` |
| Planning a development sprint | `game-sprint-plan` |
| Building a rapid prototype | `game-prototype` |
| Running a playtest session | `game-playtest` |
| Checking game balance and tuning | `game-balance-check` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlterLab-IEU/AlterLab_GameForge](https://github.com/AlterLab-IEU/AlterLab_GameForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

---
trigger: always_on
description: >
---


# Galgame Engine

Run an interactive narrative engine with a Story Director planning pass, isolated
execution passes, and an optional post-turn Memory Curator pass. During interactive
terminal play, prefer low-overhead inline state and concise player-facing output. Use
real separate model/API calls or file-backed state only when the user explicitly wants
heavier persistence/debug behavior.

Core design principle: **scene logic, character design, and character response are
separate concerns**. Characters should react from their own persona documents,
relationship state, and recent events, not from narrative convenience.

---

## Architecture Overview

Each player turn triggers a planning pass and up to **three sequential execution
passes**, each with its own prompt contract and scoped input:

```
Player Input
     │
     ├─▶ META COMMAND ROUTER  →  Handles /pause, /status, /profile, /memory, etc.
     │                           (meta commands usually do not advance story)
     │
     ├─▶ RUNTIME MODE ROUTER  →  meta | light_story | full_story
     │
     ├─▶ CONTEXT FIREWALL  →  Builds scoped inputs; removes hidden fields
     │
     ├─▶ [0] STORY DIRECTOR AI  →  Beat plan, pressure, hooks, event timing
     │                              (no prose, no character dialogue)
     │
     ├─▶ [1] SCENE NARRATOR AI  →  Environment description (sensory only)
     │
     ├─▶ [2] CHARACTER ARCHITECT AI  →  (triggers only on new/updated characters)
     │         Writes / updates Persona Document in World State
     │
     ├─▶ CONTEXT FIREWALL  →  Builds response_safe_persona per character
     │
     └─▶ [3] CHARACTER RESPONSE AI  →  Reads response_safe_persona, generates behavior
               (called once per relevant character in the scene)
                    │
                    ▼
             EDITOR PASS  →  Merges outputs, checks consistency, formats final text
                    │
                    ▼
             MEMORY CURATOR  →  Post-turn state consolidation, not player-facing
```

The **World State** is the only shared memory. Passes write to it or read filtered
slices from it; they never directly share private reasoning.

The Memory Curator is not a fourth narrator. It runs after rendering, classifies what
should become core memory, recall memory, archival memory, relationship change, or
discarded routine detail.

The Story Director is not a narrator and not a character controller. It decides pacing,
pressure, scene hooks, event timing, and whether the moment is ready for a new character
or reveal. It must not write player-facing prose, decide a character's emotions, or
force a character to confess, submit, forgive, or reveal a secret.

The Meta Command Router runs before story modules. It handles slash commands such as
`/pause`, `/status`, `/profile`, `/memory`, `/tone`, `/debug`, `/save`, `/load`,
`/newrole`, `/import-card`, and `/export-card`. Most meta commands inspect or update
state without advancing the scene; `/newrole` is a story-control command that explicitly
creates a character and advances the scene. Full command rules:
`references/meta-commands.md`.

---

## Performance Policy

Default to **interactive play mode** unless the user asks for file-backed testing,
debug traces, or persistent saves.

Rules:
- Keep `world_state` inline during ordinary play. Do not create or rewrite JSON files
  every turn unless `/save`, `/load`, `/debug`, or explicit persistence is requested.
- Do not show module traces, tool logs, file paths, state diffs, or schema details in
  normal story output. Numeric relationship stats belong only in the compact
  `GALGAME_STATE_DELTA` block, not in rendered prose.
- Load only the reference needed for the current operation. Do not read all references
  at session start.
- Use `light_story` for ambience, movement, and low-stakes actions; promote to
  `full_story` only when character agency or state changes require it.
- Batch state maintenance mentally or inline; run Memory Curator in compact form unless
  the user requests inspection.

Player-facing story output should be only the rendered scene, character behavior/dialogue,
and a decision point. Status summaries belong to `/status`; engine internals belong to
`/debug`.

For terminal play, maintain state with a compact inline delta before the story text:

```text
GALGAME_STATE_DELTA turn=<n> mode=<mode>
events: ...
characters: <id> affection +x -> <total>/100 trust +y -> <total>/100 guard +z -> <total>/100; tier <same|changed|none>
memory: ...
scene: ...
```

Keep it short and machine-readable. The totals are post-turn canonical values used by
later character responses. Do not explain the numbers in normal play. This block records
state in context without file writes; `/save` later persists the current inline state.

---

## Turn Output Contract

Every non-meta story turn must start with a compact inline state update, then the
rendered story text:

```text
GALGAME_STATE_DELTA turn=<n> mode=<light_story|full_story>
events: <event_type>(<character_ids>) importance=<0-5>
characters: <id> affection +x -> <total>/100 trust +y -> <total>/100 guard +z -> <total>/100; tier <same|changed|none>
memory: <short state memory or no_change>
scene: <short scene update>

---
Turn <n>

<player-facing narrative>

▎ <decision point>
```

Rules:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shukahub/galgame-engine](https://github.com/Shukahub/galgame-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

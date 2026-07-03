---
trigger: always_on
description: This app extends the shared design system in `docs/design-system.md`.
---

# Minelogue

This app extends the shared design system in `docs/design-system.md`.

## Design System

The Minelogue-specific design system follows the app navigation hierarchy.
Shared typography, spacing, shape, color, control, and motion primitives come
from `docs/design-system.md`. This file defines how those primitives are
applied to the Minelogue domain.

### 1. Navigation Map

The app has one persistent global command region and two primary views:

- Global Header / Command Bar
- Timeline View
- Waterfall View

Timeline and Waterfall share transcript semantics, search state, and deep-link
behavior, but their selection and pinning state must remain separate unless a
user action explicitly crosses views.

### 2. Global Header / Command Bar

Purpose: persistent session-level navigation and view control.

Contains:

- Sessions link.
- Backward and Forward transcript navigation.
- Session title and session info.
- Project path and branch metadata.
- Search toggle.
- Timeline / Waterfall switch.
- Copy link action.

Rules:

- Use shared `control-height` for command buttons.
- Use shared `ui` typography for command buttons and search toggles.
- Use shared `body` typography for session/project metadata.
- Search toggle active state uses the shared accent family.
- The command bar should remain visually quiet; avoid heavy shadows and
  decorative treatments.

### 3. Timeline View

Timeline is the structural overview of the transcript across main agent and
subagent tracks. It prioritizes spatial relationships, semantic block scanning,
and stable overlay placement.

#### 3.1. Timeline Search Shelf

Purpose: inline Timeline search controls inserted between the command bar and
the Timeline agent list.

Rules:

- Insert and remove the shelf with a bounded transition.
- The shelf uses shared search control sizing:
  - search input height: 32px.
  - search input typography: shared `ui`.
  - search navigation buttons: compact 28px controls.
- Preserve the same horizontal inset as nearby Timeline surfaces.
- Keep a compact two-row rhythm: primary search row plus options row.
- Search shelf expansion must trigger Timeline canvas and detail-window layout
  remeasurement during and after the transition.

#### 3.2. Agent List / Timeline Header

Purpose: fixed structural map of the main agent and subagents.

Rules:

- The agent list is its own horizontal scroller: it follows canvas panning,
  but horizontal gestures over the header scroll the list independently
  without moving the block canvas; the next horizontal canvas movement snaps
  the list back into alignment. Vertical gestures over the header pass
  through to the canvas. Selecting an agent jumps the canvas to that lane
  (loading its content first when needed).
- Agent cells fill their full column edge-to-edge: top, bottom, left, and right.
- Text content keeps internal padding from those edges.
- Do not use card borders for agent cells.
- Selected state uses full-cell background highlight.
- Error state uses a bottom-right exclamation icon and does not change the
  agent background color.
- The main agent uses `MAIN` and does not show a separate name or description.
- Subagents use `SUBAGENT`, show the subagent name instead of UUID, and may show
  up to two description lines before truncation.
- Model and block count sit at the bottom-left for both main and subagent cells.
- `MAIN` and `SUBAGENT` must use identical kicker typography.

#### 3.3. Timeline Canvas

Purpose: virtualized, scrollable block layout showing transcript flow by agent.

Rules:

- Timeline geometry must stay deterministic under virtualization.
- Search hit/context state must not alter block dimensions.
- Selection and active states must not alter track layout.
- The canvas should render semantic relationships without decorative grid noise.

#### 3.4. Timeline Blocks

Purpose: compact semantic markers for transcript blocks.

Typography:

- Use badge/kicker typography, not body typography.
- Recommended block label spec:

```css
font-size: 10px;
line-height: 1;
font-weight: 850-900;
letter-spacing: 0;
text-transform: uppercase;
white-space: nowrap;
```

Rules:

- Timeline block labels represent semantic kind, not message content.
- Labels should remain short, centered, and stable.
- Do not resize blocks to fit text.
- Truncate or ellipsize labels when needed.
- Strong semantic fills are appropriate because blocks are small and must scan
  quickly.

#### 3.5. Connectors / Spawn Arrows

Purpose: show parent tool calls, subagent spawning, and response relationships.

Rules:

- Spawn message starts below the line of the agent that made the call.
- Connector paths use rectangular corners.
- Connector path structure:
  - start from the calling agent's tool call.
  - route to the center of the subagent column.
  - turn downward and point to the subagent spawning message.
- Normal and highlighted connector appearances must be identical except for
  color.

#### 3.6. Detail Window Dock / Window Layout Area

Purpose: deterministic placement for live and pinned detailed message windows.

Rules:

- Detail windows are managed inside a computed "window layout area."
- The window layout area begins below the Timeline agent list plus the standard
  clearance.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WeZZard/minelogue](https://github.com/WeZZard/minelogue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->

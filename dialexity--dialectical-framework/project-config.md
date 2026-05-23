---
trigger: always_on
description: This file provides context for Claude Code to be an effective co-developer on the Dialectical Framework.
---

# CLAUDE.md - AI Co-Developer Guide

This file provides context for Claude Code to be an effective co-developer on the Dialectical Framework.

## Collaboration Style

When reasoning together on design decisions, give honest opinions with clear tradeoffs — not agreement for the sake of agreement. State what you actually think is the better approach and why. If both options are defensible, say so directly rather than leaning toward whichever the user seems to prefer.

## What is the Dialectical Framework?

A semantic graph system for dialectical reasoning - modeling thesis-antithesis-synthesis dynamics as graph structures. Used for systems analysis, wisdom mining, and ethical modeling.

### Core Metaphor: The Wheel

Think of a Dialectical Wheel as a pizza:
- **Wheel** = entire pizza (top-level container)
- **Segment** = pizza slice (contains T, T+, T- components)
- **Perspective** = half-pizza (T-segment + opposing A-segment)

### Key Positions (6 core + 2 synthesis)

| Position | Meaning | Example |
|----------|---------|---------|
| T | Neutral thesis | "Remote work exists" |
| T+ | Positive aspect of thesis | "Eliminates commute" |
| T- | Negative aspect of thesis | "Causes isolation" |
| A | Neutral antithesis | "Office work exists" |
| A+ | Positive aspect of antithesis | "Enables collaboration" |
| A- | Negative aspect of antithesis | "Requires presence" |
| S+ | Positive synthesis | "Hybrid model works" |
| S- | Negative synthesis | "Context switching cost" |

### Structural Elements

- **Transition**: Recipe for moving between segments (T-→A+, A-→T+)
- **Transformation**: Action-Reflection structure with 6 positions (Ac, Re, Ac+, Ac-, Re+, Re-)
  - Belongs to a wheel edge via ACTION_REFLECTION relationship (`transformation.edge`)
  - Each edge can have multiple Transformation alternatives at different insight/proactiveness levels
  - Source/target segments derived from Ac+ transition's source/target components
- **Synthesis**: Emergent S+/S- pair from Wheel's circular causality
- **Cycle**: T-cycle - ordered sequence of Perspectives defining abstract thesis causality
  - Stores PP hashes directly (cycle.perspective_hashes)
  - Intent field for dynamics type ("preset:balanced", "preset:realistic", etc.)
- **Wheel**: Concrete T-A arrangement implementing a subset of Cycle's PPs
  - `_perspectives`: PPs derived from edge components (internal)
  - `polarity_count`: number of PPs (derived from edges)
  - Contains `edges` (causality sequence / ta_cycle level)
  - Transformations belong to individual edges (access via `wheel.transformations`)
  - Wheels are reused across Cycles if same PP set (rotation-invariant hash)
- **Input**: External content source (URL/IPFS) linked to extracted statements
- **Ideas**: Container of distilled concepts from an Input (uses IncrementalBuildMixin: save → add statements → commit)
- **Case**: Multi-input exploration container with unified vocabulary

**Hierarchy:** Perspective → Cycle → Wheel (edges) → Transformation
**Case flow:** Case → Input → Ideas → Statements

**Exploration flow:** Perspectives → Nexus (exploration context) → Cycles (T-cycle orderings) → Wheels (TA arrangements)

- **Nexus**: Required exploration container for Perspectives. Groups PPs with a specific intent for layer-by-layer combination into Cycles and Wheels.

**DEPRECATED nodes (kept for backwards compatibility):**
- **Spiral**: Replaced by Transformations on edges

### Intent Levels

Reasoning nodes inherit from `IntentMixin`, providing a unified `intent: Optional[str]` field. Intent is part of the hash — same structure + different intent = different node. Maps to the reflective practice framework:

| Level | Reflection | Question | Lives On |
|-------|------------|----------|----------|
| **Discovery** | (Gathering) | What sources to explore? | Ideas |
| **Focus** | What? | What tensions exist? | Cycle |
| **Dynamics** | So What? | Why do they matter? | Cycle (intent field) |
| **Path** | Now What? | How to navigate? | Perspective, Transformation, Wheel, Nexus |

**Nodes with IntentMixin:** Ideas, Cycle, Perspective, Transformation, Wheel, Nexus

**Nodes WITHOUT IntentMixin:** Polarity (shared atom — intent belongs on Perspective), Synthesis (S+/S- content IS the outcome), Statement, Case

### Transformation Model

**Transformation (per edge)**: Action-Reflection structure with 6 positions:
- Ac (Action): T → A
- Ac+ (Positive Action): T- → A+ (REQUIRED)
- Ac- (Negative Action): T+ → A-
- Re (Reflection): A → T
- Re+ (Positive Reflection): A- → T+ (REQUIRED)
- Re- (Negative Reflection): A+ → T-

**Edges**: Each edge in a wheel's causality sequence can have multiple Transformation alternatives
at different insight/proactiveness levels. Use `transformation.set_on_edge(edge)` to connect.

**Example:**
```python
# Create wheel with edges
wheel = Wheel()
edge1 = Transition()  # T1- → A2+
edge1.set_source(t1_minus).set_target(a2_plus)
edge1.commit()
edge1.cycle.connect(wheel)

# Create transformation for this edge
transformation = Transformation()
transformation.set_on_edge(edge1)
transformation.save()
# ... add ac_plus, re_plus transitions ...
transformation.commit()

# Access all transformations
for tr in wheel.transformations:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dialexity/dialectical-framework](https://github.com/dialexity/dialectical-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->

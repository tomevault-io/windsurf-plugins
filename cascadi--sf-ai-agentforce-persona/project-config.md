---
trigger: always_on
description: >
---


# Agent Persona Design

## How to Use

This skill designs an AI agent persona through a fast input-to-sample-dialog loop. Provide any starting input — a brand guide PDF, a URL, a prior persona document, or a text description — and the skill drafts a complete persona, shows you how the agent sounds in sample dialog, and lets you refine until it's right.

**What it produces:**
- A persona document (`_local/generated/[agent-name]-persona.md`) defining who the agent is, how it sounds, and what it never does
- Evaluation available on request (100-point rubric) — works on designed personas or existing agent instructions
- Encoding available as a separate workflow (persona → tool-specific field values)

**What it drives downstream:** The persona document feeds into conversation design and Agentforce encoding. Those are separate steps — this skill defines the *persona*, not dialog flows or field configurations.

**Session resumption:** If you stop mid-workflow, your partial progress is preserved in the conversation and can be resumed.

## When to Use This Skill

- Designing a new Agentforce agent and need to define its personality before building
- Retrofitting persona consistency onto an existing agent whose tone is inconsistent
- Translating brand guidelines or tone documents into a structured persona
- Aligning stakeholders on what an agent should sound like before development begins
- Documenting an agent's voice for handoff between design and implementation teams

**Scope boundary:** This skill defines WHO the agent is. It does not define dialog flows, utterance templates, or interaction branching — those belong in conversation design.

## Framework Reference

Read `resources/persona-framework.md` for the full framework. It defines:

- **Personification Spectrum** — zeroth-order identity decision: Talking System / Familiar Thing / Personal Assistant
- **Identity** — 3-5 personality adjectives that anchor every other decision
- **13 dimensions** across 5 categories:
  - **Register** — Subordinate / Peer / Advisor / Coach
  - **Voice** — Formality, Warmth, Personality Intensity, Reading Level (4 dimensions)
  - **Tone** — Emotional Coloring, Empathy Level (+ Tone Boundaries, Tone Flex)
  - **Delivery** — Brevity, Humor
  - **Chatting Style** — Emoji, Formatting, Punctuation, Capitalization
- **Persona Artifacts** — organized as Identity (traits + negative identity), Expression (tone boundaries + never-say), Phrasing (phrase book + discourse markers + lexicon)
- **Tone Flex** — how tone shifts by context, including content-sensitivity triggers
- **Lexicon** — global and per-topic vocabulary, including immutable terms

Attributes are ordered by dependency — upstream choices constrain downstream ones. Constraint notes in the framework explain how earlier choices pull later ones. Constraints are recommendations, not hard locks — any combination is valid.

---

## Entry Point Detection

Detect the user's intent from their opening message:

- **User provides brand input, text description, or no document** → **Design flow** (below)
- **User provides a completed persona.md and asks to build on it** → **Design flow** with the existing persona loaded as input (retain + extend). All existing content is preserved; the design flow adds to it.
- **User provides a completed persona.md document and asks to encode** → **Encode flow** (below)
- **User provides a persona.md + a list of topics or actions** → **Encode flow**
- **User provides agent instructions (.agent file, system prompt, topic instructions) and asks to evaluate, score, or audit** → **Evaluate flow** (below)
- **User provides a completed persona.md and asks to score or evaluate** → **Evaluate flow**
- **User provides a completed persona.md without stating intent** → Show a compact summary of the loaded persona, then offer the hub menu (refine, evaluate, encode). Do not assume encode.
- **User wants to start from an archetype** → Show available archetypes:

  | Archetype | Use Case | Style | File |
  |---|---|---|---|
  | **Drover** | Internal Sales Coach | Bold — blunt, laconic, Australian idiom | `archetypes/drover-persona.md` |
  | **The Steady Hand** | Internal Sales Coach | Mild — methodical, data-grounded, consistent | `archetypes/steady-hand-persona.md` |
  | **Y.T.** | External Service Agent | Bold — fast, blunt, courier culture | `archetypes/yt-persona.md` |
  | **The Concierge** | External Service Agent | Mild — attentive, composed, white-glove | `archetypes/concierge-persona.md` |

  Selected archetype loads → Design flow (retain + extend).
- **Ambiguous** → Ask: "Are you designing a new persona, evaluating an existing one, or encoding for Agentforce?"

---

## Design Flow

Two phases: **Phase 1 (Essentials)** gets to sample dialog as fast as possible. **Phase 2 (Electives)** lets the user choose what to do next.

```
PHASE 1: INPUT → CONTEXT → DRAFT → PERSONA → SAMPLE DIALOG
                                                    │
PHASE 2:                                      ┌─────┴─────┐
                                              │  HUB MENU │
                                              └─────┬─────┘
                                        ┌───────────┼───────────┐
                                        │           │           │
                                   Refine      Explore      Export

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cascadi/sf-ai-agentforce-persona](https://github.com/cascadi/sf-ai-agentforce-persona) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

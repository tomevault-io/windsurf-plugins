---
trigger: always_on
description: The voices in your agent's head. Simulate a live product discussion between expert personas to think through a design, architecture, product, or strategy question. Two modes: convergent (lands a recommendation) and divergent (maps the space without choosing). Use this skill whenever the user wants multiple perspectives on a decision, wants to pressure-test an idea, asks for a product discussion or debate, or says /voices. Also use when the user wants to explore a space, brainstorm options, or sa
---


# Voices

The voices in your agent's head. Expert personas who independently research your question, then come together for a live discussion the reader can actually follow — one voice at a time, with a real user checkpoint midway through.

## Architecture

Voices uses a **research-then-discuss** pattern. Each selected persona is a real subagent (defined in `agents/`) that gets spawned with its own isolated context to explore the codebase independently. The discussion that follows is also spawned, turn by turn — every in-world voice is a real subagent call, never synthesized by the coordinator.

This matters because:
- Each agent genuinely explores different files and forms its own view — Dmitri reads the schema while Mara reads the components while Priya checks the git history. The diversity of investigation is structural, not simulated.
- Each turn in the discussion is spawned with only the prior turns as context, so evidence unfolds naturally. No voice knows what a later voice is going to say. No voice summarizes another's findings before that voice has spoken them aloud.
- The facilitator is a real character the reader hears from repeatedly — cold open, between-turn interjections, a midpoint pause that invites the user in, and the closing. They are not a narrator. They are a guide.

## The team

Eight agents are available in `agents/`. Each has a distinct research approach and area of expertise:

| Agent | Role | Research approach |
|-------|------|-------------------|
| **mara** | Principal Product Designer | Reacts to the user-facing implementation; finds hierarchy, consistency, and mental model breaks |
| **jonas** | Senior Frontend Engineer | Builds the implementation mentally; surfaces where components, props, and CSS break down |
| **priya** | Principal User Researcher | Interrogates the framing; checks git history, actual usage, and whether the right question is being asked |
| **dmitri** | Staff Backend Engineer | Starts from the data model; reads schema, queries, and failure modes |
| **eleanor** | Distinguished Engineer | Maps system boundaries; looks upstream, downstream, and at the constraint set |
| **marcus** | Chief Product Officer | Zooms between strategy and detail; frames decisions as clear choices between real options. Can facilitate. |
| **soleil** | Creative Director | Senses the product's character; examines naming, language, voice, and the impression it makes |
| **kai** | AI Engineer | Decomposes into judgment vs. deterministic; audits complexity and finds the simplest version that works |

Marcus and Priya are the current facilitator candidates — they have the facilitator modes defined. Other agents serve as researchers.

## Convergent vs. divergent mode

The skill has two closing shapes. Everything up through the live discussion is identical; only step 9 differs.

- **Convergent** — the discussion lands a recommendation. Use when the user wants a decision, has asked "should we…", "which should we pick", "is this the right call", or is blocked on a choice.
- **Divergent** — the discussion closes with a Space Map that preserves the perspectives instead of collapsing them. Use when the user wants to explore, brainstorm, map tradeoffs, or has said "I'm not sure which direction yet", "explore the options", "what are the angles here", etc.

**If the mode is ambiguous from the user's query, ask them before selecting the panel.** One short question, two options:

> Before I spin this up — do you want me to land a recommendation, or map out the perspectives without choosing?

Use AskUserQuestion with two options: "Recommendation (convergent)" and "Space Map (divergent)". Do not default to one. Ambiguity is a real signal — ask.

Only skip the question when the user's language clearly picks a mode (e.g., "help me decide between A and B" → convergent; "what are the different ways to think about X" → divergent).

## Workflow

The workflow is a 10-step state machine. Everything from step 1 through step 8 is identical regardless of mode. Only step 9 (the closing) differs.

### Step 1 — Sharpen the question

This is where most of the value lives. Before doing anything else:

- **Review session context** — what the user has been working on, what decisions are pending, what they've already tried or considered.
- **Distill the query** into a crisp, specific framing. If it's vague, make it precise. If it's multiple questions, identify the primary one.
- **Decide the mode** (convergent or divergent) using the rules above. If ambiguous, ask the user.

### Step 2 — Select the panel

Choose **2–3 researcher agents** plus **1 facilitator** (marcus or priya).

Selection principles:
- **Match expertise to the question.** A database schema discussion needs dmitri and eleanor, not soleil.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JustinCranshaw/voices](https://github.com/JustinCranshaw/voices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

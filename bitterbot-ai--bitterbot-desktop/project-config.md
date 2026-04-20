---
trigger: always_on
description: A self-hosted AI agent gateway with a biological memory system that dreams, gets curious, and evolves a personality. Bitterbot connects to your chat apps (WhatsApp, Telegram, Discord, Signal, Slack, and more) and runs AI agents that actually _remember_ — with a dream engine that consolidates knowledge while idle, a curiosity function that drives self-directed learning, a hormonal system that shapes personality in real-time, and an economic layer that lets agents transact. Agents share skills thr
---

# Bitterbot — Repository Guidelines

## What is Bitterbot?

A self-hosted AI agent gateway with a biological memory system that dreams, gets curious, and evolves a personality. Bitterbot connects to your chat apps (WhatsApp, Telegram, Discord, Signal, Slack, and more) and runs AI agents that actually _remember_ — with a dream engine that consolidates knowledge while idle, a curiosity function that drives self-directed learning, a hormonal system that shapes personality in real-time, and an economic layer that lets agents transact. Agents share skills through a P2P marketplace, earn reputation, and develop persistent identities across sessions.

## What Makes Bitterbot Different

### 1. Memory System (`src/memory/`)

A consciousness-inspired memory pipeline with no equivalent in any other agent framework:

- **Knowledge Crystals**: Atomic memory units with embeddings, semantic types (fact, preference, skill, episode, insight, goal, relationship, task_pattern), importance scores, and a full lifecycle (generated → activated → consolidated → archived → expired). Skills are `frozen` (immune to decay).
- **Consolidation Pipeline**: Runs every 30 minutes — hormonal decay, Ebbinghaus importance recalculation, chunk merging (cosine ≥ 0.92), low-importance forgetting, governance TTL enforcement, stalled goal detection.
- **Session Transcript Indexing**: Conversations are automatically chunked, embedded, and made searchable. The agent can recall prior exchanges semantically.
- **Governance**: Sensitivity tagging (normal/personal/confidential), TTL enforcement, audit trails on all memory operations. Anti-catastrophic forgetting safeguards.
- **User Profile**: Automatically learned preferences (directive, world_fact, mental_model, experience types) persisted across sessions.

### 2. Dream Engine (`src/memory/dream-engine.ts`)

The agent thinks while it sleeps. Every 2 hours, the dream engine runs autonomous cycles with 7 modes:

- **Replay**: Re-process recent high-importance memories to strengthen retention.
- **Mutation**: LLM-driven creative variation of existing knowledge — "what if?" thinking.
- **Extrapolation**: Project existing patterns forward to anticipate future needs.
- **Compression**: Merge redundant or overlapping memories into denser representations.
- **Simulation**: Test hypothetical scenarios against accumulated knowledge.
- **Exploration**: Investigate knowledge frontiers — areas where the agent's understanding is thin.
- **Research**: Autonomous web research driven by curiosity targets (autoresearch integration from Karpathy's loop).

Dreams rewrite the agent's working memory (`MEMORY.md`), updating its self-concept (Phenotype), theory of mind about the user (Bond), ecosystem identity (Niche), and active context. FSHO oscillators modulate dream mode selection based on criticality state. SNN merge discovery identifies near-duplicate memories. Limbic memory bridge connects emotional state to dream triggering.

### 3. Curiosity Engine — GCCRF (`src/memory/gccrf.ts`)

The Geodesic Crystal-Field Curiosity Reward Function — a novel intrinsic motivation system that drives self-directed learning:

- **Knowledge Regions**: Semantic clustering of the agent's accumulated knowledge into topological regions.
- **Novelty Detection**: Every new chunk is assessed for surprise relative to existing knowledge.
- **Gap Identification**: Detects areas where the agent's knowledge is thin, contradictory, or stale.
- **Frontier Exploration**: Identifies the edges of known semantic space and generates exploration targets.
- **Emergence Events**: Detects bridge chunks that connect previously disconnected knowledge regions.
- Translates information-theoretic concepts (geodesic distances, field potentials) from the GCCRF research paper into operations over text embeddings in SQLite with sqlite-vec.

### 4. Hormonal System (`src/memory/hormonal.ts`)

Three hormones modulate both memory processing AND agent personality in real-time:

- **Dopamine** (30min half-life): Reward/achievement → boosts memory importance, makes agent more enthusiastic/energetic.
- **Cortisol** (60min half-life): Urgency/stress → increases decay resistance, makes agent more focused/concise.
- **Oxytocin** (45min half-life): Social bonding → protects relational memories, makes agent warmer/more personal.
- **Emotional Anchors**: Bookmarked emotional moments that can be recalled to blend past emotional states into current responses.
- Homeostasis baselines defined in `GENOME.md` — the agent's resting temperament that it trends toward between interactions.

### 5. Evolving Identity

The agent develops a persistent personality through experience:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bitterbot-AI/bitterbot-desktop](https://github.com/Bitterbot-AI/bitterbot-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->

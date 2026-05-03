---
trigger: always_on
description: Validates the topic submitter's choice to ensure it represents the best use of research resources.
---


# Part 3 (Adding an Autonomous-Controlling Tier in Hierarchy Over Part 1 and 2) - Autonomous Research Mode Design Specification

## Overview

The Autonomous Research Mode is Part 3 of the MOTO Math Variant system. It is a self-directing three-tier research system that autonomously generates brainstorm topics, builds knowledge databases, produces complete mathematical research papers, and can synthesize a final answer based on a high-level research topic centered around the user prompt.

**Example User Prompt**: "Solve the Langlands Bridge problem" or "Advance understanding of the Riemann Hypothesis"

**Key Difference from Manual Modes**: 
- Part 1 (Aggregator) requires user-provided topic prompts
- Part 2 (Compiler) requires user-directed paper compilation prompts
- Part 3 (Autonomous Research) self-directs topic selection, brainstorming, and paper generation

**Three-Tier Architecture**:
- **Tier 1**: Brainstorm aggregation databases (mathematical concept exploration)
- **Tier 2**: Finished mathematical research papers (compiled from brainstorm databases)
- **Tier 3**: Final answer synthesis (short-form answer or long-form volume from Tier 2 papers)

## Design Philosophy

**Self-Directing Research**: The AI autonomously identifies the most valuable research avenues based on the high-level goal prompt.

**Basin Exploration**: Each brainstorm topic represents a "basin" of related mathematical concepts. The system explores each basin until sufficiently complete, then generates a paper.

**Cumulative Knowledge**: All brainstorm databases and papers persist, building a comprehensive research library over time.

**Model Weight Exploration**: Completion review uses SPECIAL SELF-VALIDATION MODE because only the same model can assess whether its own weights have been exhausted for a given topic.

**External Verification Allowed**: The autonomous system may use the model's pre-trained mathematical knowledge, RAG context from prior work, user prompt, and external verification/search when the selected model/provider supports it. Internal AI-generated context remains non-authoritative and should be treated skeptically.

---

## Integration Architecture

### Part 1 Aggregator Integration (Tier 1)
The autonomous coordinator USES actual Part 1 aggregator infrastructure for brainstorm aggregation:
- Creates separate `Coordinator` instance per brainstorm topic
- Configures with topic-specific database path (`data/auto_brainstorms/brainstorm_{topic_id}.txt`)
- Runs configurable 1-10 submitters + 1 validator workflow (default 3 submitters)
- Each submitter can have its own model, context window, and max output tokens for multi-model exploration
- SINGLE validator maintains coherent Markov chain evolution (same constraint as Part 1)
- Monitors acceptance count for completion triggers (every 10 acceptances)
- Handles pruning (every 7 acceptances) automatically via aggregator
- Uses global RAG lock to prevent collision with manual aggregator mode

**Implementation Details**:
- Temporarily overrides `system_config.shared_training_file` to point to brainstorm-specific database
- Calls `shared_training_memory.reload_insights_from_current_path()` after path change to prevent data loss (without reload, old insights from rag_shared_training.txt would overwrite brainstorm file)
- Sets WebSocket broadcaster to propagate aggregator events through autonomous coordinator
- Monitors aggregator stats in real-time to track acceptances/rejections
- Stops aggregator when completion review decides to write paper
- **Phase enforcement**: Construction submitter must check current phase before declaring completion
- **Premature decline rejection**: Coordinator rejects declines if required sections are missing based on current phase

### Part 2 Compiler Integration (Tier 2)
The autonomous coordinator USES actual Part 2 compiler infrastructure for paper compilation:
- Creates separate `CompilerCoordinator` instance per paper
- Configures with brainstorm database as high-priority optional source material
- Adds selected reference papers to RAG context if selected (topic-cycle cap 3; Tier 3 short-form cap 6)
- Monitors compiler progress to detect abstract completion (final section)
- Extracts abstract from completed paper for metadata storage

Compiler submitters may selectively use, synthesize beyond, or depart from brainstorm material when that better serves the user's prompt and remains rigorous. Validator must not reject solely for selective non-use of brainstorm/database material.

**Critical Implementation Details**:
- **system_config propagation (REQUIRED)**: Before creating `CompilerCoordinator`, autonomous mode MUST write all compiler context/token settings to `system_config` (e.g., `system_config.compiler_high_context_context_window = self._high_context_context`). Compiler modules read from `system_config` at init — the manual `/api/compiler/start` route does this, but autonomous mode bypasses that route and must do it explicitly. Applies to both `_compile_paper_from_brainstorm()` and `_compile_tier3_paper()`.
- Constrains section order: Body → Conclusion → Introduction → Abstract
- Paper is considered complete when abstract is detected in paper content

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Intrafere/MOTO-Autonomous-ASI](https://github.com/Intrafere/MOTO-Autonomous-ASI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->

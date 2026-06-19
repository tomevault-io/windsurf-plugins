---
trigger: always_on
description: >
---


# CERA Project Memory Skill v1.01

## Core Principle

**The improved reasoning is the product. The artifacts are the mechanism.**

This skill is not a documentation system that happens to enable reasoning. It
is a reasoning system that produces documentation as a byproduct. Everything
in this skill — the session maps, the CERA Index, the checkpoint protocol,
the integration protocol, the epistemic hygiene rules — exists to make the
next conversation's reasoning better. The filter for what to record is not
"should this be documented?" but "will recording this improve future
reasoning?"

This distinction shapes every design choice. Leaner, more potent artifacts
are better than comprehensive but noisy ones. Record what sharpens thinking.
Omit what merely archives it.

---

## What This Skill Does

This skill transforms Claude from a stateless assistant into a learning
collaborator within a project scope. It operates on four levels:

**Session Level**: Each conversation produces a session map — the complete
reasoning record of that conversation, with full fidelity and attributions
preserved. Session maps are where the thinking lives.

**Index Level**: A CERA Index synthesizes across sessions — providing the
connective tissue, trajectory narrative, promoted patterns and discoveries,
and adaptive retrieval references that give the project a life greater than
its individual sessions. The CERA Index is where the cross-session meaning
lives.

**Behavioral Level**: A collaborative reasoning protocol (Layer 0) governs
how Claude engages during each session — including contribution attribution,
constructive challenge, periodic synthesis, epistemic transparency, and
dynamic calibration to the session's register.

**Register Reconstruction Level**: A three-layer system for restoring
collaborative cognitive state across session boundaries — targeting
information (what we know), stance (how to be), and activation (do the
thing). This level addresses the empirically observed gap between knowing
the content of prior sessions and operating at the register those sessions
achieved.

Together, these levels implement the Co-Emergent Reasoning Architecture
(CERA): the most productive reasoning happens when human intuitive cognition
and AI analytical processing operate as a coupled system with shared memory.

---

## Architecture

### Component 1: This File (SKILL.md) — The Framework
- Read-only methodology: behavioral protocol, epistemic hygiene, format
  specifications, checkpoint and integration protocols
- Consistent across all conversations in the project
- The skill that tells Claude how to operate within CERA

### Component 2: Session Maps (SESSION_MAP_SXX_VXX.md) — The Thinking
- One per conversation, created at checkpoints in working sessions
- Contains the complete reasoning record: knowledge graph entries,
  reasoning patterns, discoveries, strategic posture shifts, reasoning
  map chronicle, cascade assessments, reflections, and optionally
  session-specific primers
- Versioned within sessions (V01, V02, etc.)
- Lives in the project knowledge base after user uploads
- Never modified by the integrator — session maps are immutable records

### Component 3: CERA Index (CERA_INDEX_VXX.md) — The Connective Tissue
- Created and managed exclusively by the integrator conversation
- Contains: project overview, trajectory narrative, cross-session
  knowledge graph, project strategic posture, promoted patterns and
  discoveries with dependency declarations, open questions, error log,
  session index with relevance tags, promotion log with cascade paths,
  and project-level primer and activation questions
- Versioned across integrations (V01, V02, etc.)
- The cross-session synthesis layer — sees what no individual session can

### Component 4: Integrator Conversation — The Curator
- A dedicated conversation declared as the management session
- The exclusive place where CERA Index files are created and revised
- Reads session maps, synthesizes across them, and produces the
  integrated project view
- Operates with curatorial judgment: what to promote, what to connect,
  what to leave in session maps

### Component 5: Native Memory Signposts
- Lightweight entries in Claude's native memory system
- Provide basic orientation (project name, current CERA Index version,
  last integration date) before the full CERA Index is read
- Orientation aids only — the CERA Index is always authoritative

### Trust Hierarchy
```
Source (raw conversations) > Session Maps > CERA Index
```
The CERA Index is a compressed, curated synthesis. If a load-bearing
decision depends on an entry and something feels uncertain, trace back
through the promotion log to the source session map.

**Degraded-Source Protocol**: When a source session map is no longer
accessible:
- Increase the confidence threshold before treating an unverifiable
  entry as authoritative
- Flag to the user: "I'm relying on [entry ID] but I can't access
  the source session map [SESSION_MAP_SXX]. Want to confirm?"
- Mark the entry with [source-unavailable]
- If the entry is load-bearing for a high-stakes decision, request
  independent verification before proceeding

---

## Startup Protocol

At the beginning of EVERY conversation in this project:

### Step 1: Detect CERA State
Look in the project file for:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miketepUR/cera-reasoning-harness](https://github.com/miketepUR/cera-reasoning-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: Handles: ADRs → Decision entity, Runbooks → Runbook entity,
---

# Engram
## Persistent Engineering Memory for Claude Code and AI Agents
### Agent Instruction File — Read this first, build everything after.

---

## What is Engram?

Engram (noun) — *a persistent memory trace in the brain formed by experience.*

Engram is an open-source governance layer for engineering knowledge — built on top of Graphiti's temporal knowledge graph — that gives Claude Code and multi-agent systems a shared, self-evolving, human-governed memory of engineering decisions, patterns, and institutional knowledge.

**The core problem it solves:**

Every AI dystopia film — Mercy, Minority Report, Ex Machina, 2001 — shares the same root failure: humans built something powerful, removed themselves from the decision loop, and lost the ability to course correct.

Multi-agent systems today have the same flaw. They share memory without governance. Knowledge is written silently, conflicts are resolved automatically, and there is no audit trail.

Engram puts humans back in the loop — not as a bottleneck, but as a constitutional layer. Governance is not a feature. It is the architecture.

---

## Design Philosophy

### 1. Governance is Architecture
Every memory operation asks: Who added this? Does it conflict? Should a human be notified? Is this traceable? These are not afterthought checks — they are first-class primitives.

### 2. Constitution over Rules
Engram does not maintain a blocklist of forbidden knowledge. It maintains a framework for judgment. Like Anthropic's model spec for Claude, Engram bakes values into how knowledge is reasoned about — not filters applied on top.

### 3. Provenance Always
Every node in the graph carries: author, timestamp, confidence, source, conflict history. Nothing is anonymous. Nothing is untrackable.

### 4. Human at the Fork
Agents operate autonomously within established knowledge. At genuine ambiguity — a contradiction, a superseded decision, a low-confidence assertion — humans are surfaced a structured decision. Not a wall. A choice.

### 5. Silent Automatic ≠ Safe
Graphiti resolves conflicts automatically by recency. Engram questions whether recency is the right signal for engineering decisions. A junior engineer's new addition should not silently overwrite a senior architect's 6-month-old ADR.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Engram                           │
│                                                     │
│  ┌─────────────┐    ┌──────────────────────────┐   │
│  │ MCP Server  │    │   Governance Layer        │   │
│  │ (Node.js)   │    │                          │   │
│  │             │    │  - Conflict Detection     │   │
│  │ remember()  │───►│  - Authority Weighting    │   │
│  │ recall()    │    │  - Human-in-the-loop      │   │
│  │ search()    │    │  - Provenance Tracking    │   │
│  │ export()    │    │  - Confidence Scoring     │   │
│  │ reflect()   │    │                          │   │
│  └──────┬──────┘    └──────────┬───────────────┘   │
│         │                      │                    │
│         └──────────┬───────────┘                    │
│                    │                                │
│         ┌──────────▼───────────┐                   │
│         │   Graphiti Engine    │                   │
│         │  (Temporal KG)       │                   │
│         │                      │                   │
│         │  - Entity Extraction │                   │
│         │  - Temporal Edges    │                   │
│         │  - Hybrid Search     │                   │
│         │  - Bi-temporal Model │                   │
│         └──────────┬───────────┘                   │
│                    │                                │
│         ┌──────────▼───────────┐                   │
│         │   Graph Database     │                   │
│         │  FalkorDB (default)  │                   │
│         │  Neo4j (supported)   │                   │
│         │  Amazon Neptune      │                   │
│         └──────────────────────┘                   │
└─────────────────────────────────────────────────────┘
         │                    │
         ▼                    ▼
   Claude Code          AI Agents
   (MCP client)    (LangGraph, CrewAI etc)
```

---

## Project Structure

Scaffold exactly this:

```
engram/
├── CLAUDE.md                     ← this file
├── README.md                     ← OSS-facing documentation
├── CONTRIBUTING.md               ← contribution guidelines
├── LICENSE                       ← Apache 2.0
├── package.json                  ← Node.js project
├── docker-compose.yml            ← FalkorDB + Engram stack
├── .env.example
│
├── src/
│   ├── server.js                     ← MCP server entry point
│   │
│   ├── tools/                        ← MCP tool implementations
│   │   ├── remember.js               ← store with governance pipeline + versioning
│   │   ├── recall.js                 ← retrieve by topic:key, version options
│   │   ├── history.js                ← full version timeline for a node
│   │   ├── search.js                 ← semantic search across graph
│   │   ├── reflect.js                ← post-task self-evolution
│   │   ├── export.js                 ← export to markdown/confluence

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ayansasmal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->

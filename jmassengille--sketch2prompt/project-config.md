---
trigger: always_on
description: > "I build tools that help humans define systems so AI can stop guessing."
---

# sketch2prompt

## Purpose

> "I build tools that help humans define systems so AI can stop guessing."

Sketch2Prompt is a **system definition tool** for software architecture. It transforms a builder's mental model into an explicit, structured specification that **constrains** downstream AI behavior.

The core problem it solves: **underspecified intent**. AI coding agents hallucinate structure, invent inconsistent conventions, and degrade coherence over time—not because models are weak, but because they're given ambiguous input.

### What Sketch2Prompt IS

- A **semantic boundary-setting tool** that defines what exists, why it exists, and where responsibilities lie
- An **intent-freezing mechanism** that captures architectural decisions before entropy sets in
- A **constraint generator** that produces specifications AI agents can reliably follow

### What Sketch2Prompt IS NOT

- A diagramming tool for documentation
- An architecture or code generator
- A framework prescriber or implementation enforcer
- A performance optimizer or correctness validator

**Its job ends once structure and intent are clear.** Everything else is downstream.

### Success Criteria

Sketch2Prompt succeeds when:

- AI agents stop guessing architecture
- Follow-up prompts become **shorter**, not longer
- Refactors feel safer instead of scarier
- New features slot into existing structure instead of creating sprawl
- Output feels like: *"This system already has rules."*

### Philosophical Stance

- Humans are better at defining intent
- Agents are better at executing within constraints
- Systems degrade without explicit boundaries

**Freeze intent early**, before entropy sets in.

### What This Is NOT (Important)

Sketch2Prompt is **not a magic-box SaaS**. It doesn't generate code or claim to build your app for you.

It's a **context-engineering bootstrap** — a professional-grade starting point with best practices baked in. Developers still build their own tool; they just start with guardrails instead of a blank slate.

### Core Guardrails (What the Output Provides)

| Guardrail | What It Prevents |
|-----------|------------------|
| **Version anchoring** | Hallucinated/outdated package versions — the #1 cause of AI code rot |
| **Library-first policy** | Custom solutions when existing dependencies solve the problem |
| **Repo structure** | Spaghetti file organization, unclear boundaries |
| **Modularity constraints** | God functions, 1000-line files, deep nesting |
| **START.md bootstrap** | AI proceeding without confirming user intent |

These aren't suggestions — they're the foundation that makes AI-assisted development actually work.

---

## ⚠️ Critical Distinction: Dev Environment vs Output Artifacts

This project has a **meta-level complexity**: we're building an AI-instruction generator while working within an AI-instructed environment. These two contexts must never be conflated.

### Dev Environment (This Workspace)

Files that govern agent behavior **in this codebase**:

| File | Purpose |
|------|---------|
| `~/.claude/CLAUDE.md` | Global user instructions |
| `./CLAUDE.md` | Project-specific instructions (this file) |
| `.claude/STATUS.md` | Current development state |
| `.claude/phases/*.md` | Implementation phase guides |

**These are for the agent working on sketch2prompt itself.**

### Output Artifacts (Generated for Downstream Users)

Files that sketch2prompt **generates** for a builder's new project:

| File | Purpose |
|------|---------|
| `[OUTPUT] START.md` | Bootstrap protocol with confirmation gates |
| `[OUTPUT] PROJECT_RULES.md` | System constitution for downstream agent |
| `[OUTPUT] AGENT_PROTOCOL.md` | Workflow guidance for downstream agent |
| `[OUTPUT] specs/*.md` | Component specifications (Markdown + XML tags) |
| `[OUTPUT] diagram.json` | Re-import capability |

**These are for a different agent (Cursor, Windsurf, etc.) working on a different project that doesn't exist yet.**

### Naming Convention

When discussing output artifacts, always use the `[OUTPUT]` prefix or terms like "generated", "blueprint", or "downstream":

- ✓ "The `[OUTPUT] PROJECT_RULES.md` should include..."
- ✓ "The generated AGENT_PROTOCOL needs..."
- ✓ "Downstream agents read specs/*.yaml to..."
- ✗ "CLAUDE.md should tell agents..." (ambiguous — which one?)

### Subagent Guidance

When delegating to subagents, explicitly state:

> "You are modifying **generators** in `src/core/template-generator/` that produce output for downstream projects. The output files (`PROJECT_RULES.md`, `AGENT_PROTOCOL.md`, `specs/*.yaml`) are NOT instructions for you — they are templates that will guide a different AI assistant in a different codebase."

---

## Output Format

The primary deliverable is structured specifications that AI coding assistants (Cursor, Windsurf, Claude) can consume:

1. **Architecture Overview** - Components grouped by type with clear responsibilities
2. **Responsibility Boundaries** - What exists, what's allowed, what's forbidden
3. **Build Order** - Recommended implementation sequence by component type
4. **Component Guidance** - Per-component notes and watch-fors


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmassengille/sketch2prompt](https://github.com/jmassengille/sketch2prompt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

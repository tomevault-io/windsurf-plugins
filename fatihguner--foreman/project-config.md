---
trigger: always_on
description: Foreman is an open-source, AI-powered strategic advisor for entrepreneurs. It uses a layered architecture of Skills, Agents, Hooks, Memory, Diagnostics, Playbooks, Commands, and Output Templates to deliver contextual, framework-driven guidance. The project lives at foreman.sh and ships as a public GitHub repository.
---

# Foreman — Project Context

Foreman is an open-source, AI-powered strategic advisor for entrepreneurs. It uses a layered architecture of Skills, Agents, Hooks, Memory, Diagnostics, Playbooks, Commands, and Output Templates to deliver contextual, framework-driven guidance. The project lives at foreman.sh and ships as a public GitHub repository.

## Architecture Overview

```
Entrepreneur Input
    |
    ├→ [Commands] — Structured input: /apply, /diagnose, /run, etc.
    └→ [Hooks] — Natural language input: pattern matching → intent classification
            |
       [Orchestrator Agent] — Central brain, routes everything
            |
            ├→ [Memory Agent].read() — Retrieve entrepreneur context (5-layer system)
            |
            ├→ [Diagnostic Agent] — Triage: symptom → targeted questions → root cause
            |
            ├→ [Skill Executor Agent] — Apply framework to entrepreneur's context
            |
            ├→ [Playbook Runner Agent] — Run multi-step skill chains with checkpoints
            |
            ├→ [Output Agent] — Format results for target audience
            |
            └→ [Memory Agent].write() — Persist results and decisions
```

## System Layers

### Core Content Layers

1. **Skills** (`.claude/skills/`) — 158 self-contained `.md` files across 12 categories, each teaching one business framework, method, or practice. Every skill works standalone and can be invoked by an agent. Derived from 12 source collections.
2. **Output Templates** (`.claude/output-templates/`) — 48 fill-in-the-blank professional documents across 5 audiences: `investor/` (10), `board/` (7), `team/` (13), `self/` (10), `client/` (8).
3. **Diagnostics** (`.claude/diagnostics/`) — 20 triage systems (12 broad + 8 granular) that identify root causes through structured questioning. Each routes to specific skills, playbooks, and templates.
4. **Playbooks** (`.claude/playbooks/`) — 20 multi-step recipes (12 core + 8 extended) that chain skills sequentially with checkpoints and decision points.

### Orchestration Layers

5. **Hooks** (`.claude/hooks/`) — 17 trigger definitions (8 high-priority, 8 medium/low, 1 research) that classify natural-language input and route to diagnostics, skills, or playbooks.
6. **Agents** (`.claude/agents/`) — 6 AI agent definitions: orchestrator (central brain), diagnostic (triage), skill-executor (framework application), playbook-runner (multi-step conductor), output (formatting), memory (context persistence).
7. **Memory** (`.claude/memory/`) — 5-layer persistence system: identity (yearly), company (monthly), history (append-only), active (weekly), session (ephemeral). Schema + YAML templates.
8. **Commands** (`.claude/commands/`) — ~45 commands across 13 command files: navigation (7), execution (5), memory (8), playbook (5), output (3), meta (5).

### Tooling & Modes

9. **Scripts** (`scripts/`) — 21 utility scripts: validation (7), content creation (5), analysis (3), maintenance (4), community (2).
10. **Solo Mode** (`.claude/solo-mode/`) — Complete solopreneur adaptation layer: SOLO.md (master instruction), skill relevance scoring (158 skills), audience remapping (48 templates), diagnostic/playbook/hook adaptations (56 items). Activated via `/solo`.
11. **Stoic Mode** (`.claude/stoic-mode/`) — Philosophical depth layer that frames all system responses through Stoic principles (dichotomy of control, cardinal virtues, premeditatio malorum, amor fati). Does not change WHAT is delivered — changes HOW it is framed. Activated via `/stoic on`. Can combine with Solo Mode.
12. **Language Mode** (`.claude/language-mode/`) — Complete output language switch. All responses delivered in the specified language while internal processing remains English. Supports any language the model speaks fluently. Activated via `/language [code]`. Persists across sessions. Combines with Solo and Stoic modes.
13. **Industry Packs** (`.claude/industry-packs/`) — Sector-specific overlay system. Each pack adds benchmarks, skill overlays, diagnostic rules, and template adaptations for a specific industry. 9 packs: SaaS, Marketplace, E-Commerce, Fintech, AI/ML, HealthTech, EdTech, D2C/Consumer, Agency/Consulting. Each pack contains 4 YAML files (benchmarks, skill-overlays, diagnostic-rules, templates). Activated automatically when `memory.company.sector` matches a pack.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fatihguner/foreman](https://github.com/fatihguner/foreman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

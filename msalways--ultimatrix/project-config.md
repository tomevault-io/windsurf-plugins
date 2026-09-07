---
trigger: always_on
description: - **1761 tests (169 files), clean tsup build (ESM 1.61MB + CJS 1.63MB + DTS)**
---

## Ultimatrix v8 — Intelligence-Augmented Security Researcher

### Status
- **1761 tests (169 files), clean tsup build (ESM 1.61MB + CJS 1.63MB + DTS)**
- **318 source files**, zero test failures
- **Dual engine**: Legacy supervisor (v6/v7) + OODA solver engine (v8)
- **Council engine**: Parallel debate with structured typed outputs (no regex/text parsing)
- **56 skills** (10 domains), knowledge-based, not payload lists
- **Skill-driven tool filtering**: Skills declare toolRefs in YAML frontmatter, tools filtered per-agent
- **Human-in-the-Loop**: Browser visibility, action capture, session storage, flow reproduction
- **FIX-PLAN v8.2 COMPLETED**: All root-cause fixes implemented and verified
- **Council root-cause rewrite COMPLETED**: All regex removed, structured typed fields at all seams
- `@mastra/core` ^1.42.0, `playwright` ^1.52.0, `zod` ^4.0.0, `next` ^15.5.19

### Architecture — Dual Engine + Council

```
                    ┌──────────────────────┐
                    │   Engine Selector    │ ← config.engine: 'legacy' | 'solver'
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                 ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
    │  Legacy      │  │  Solver      │  │  Council         │
    │  Supervisor  │  │  Engine      │  │  Engine          │
    │  (Phase 1-5) │  │  (OODA)      │  │  (parallel       │
    │              │  │              │  │   debate)        │
    │  observe →   │  │  REASON →    │  │                  │
    │  learn →     │  │  EXPLORE →   │  │  debateOnce()    │
    │  attack →    │  │  CONCLUDE →  │  │  4 LLM members   │
    │  loop        │  │  loop        │  │  parallel speaks  │
    └──────────────┘  └──────────────┘  └──────────────────┘
              │                │                 │
              └────────────────┼────────────────┘
                               ▼
                     ┌──────────────────────┐
                     │  Session Runner      │ ← src/session.ts
                     │  (CoreServices)      │   ExecutionStrategy interface
                     └──────────┬───────────┘
                               │
                    ┌──────────┴───────────┐
                    ▼                      ▼
           ┌──────────────┐      ┌──────────────────┐
           │  Skills Lib  │      │  Skill-Tool      │
            │  (56 skills) │      │  Filter          │
           │  YAML meta   │      │  core tools      │
           └──────────────┘      │  always included │
                                 └──────────────────┘
```

### Execution Core (`src/core/`)

Unified interface for all engines. Each engine implements `ExecutionStrategy`.

| Module | Location | Purpose |
|--------|----------|---------|
| **Types** | `src/core/types.ts` | `ExecutionStrategy`, `StrategyContext`, `CoreServices`, `EnginePreset`, `RunResult` |
| **ToolPack** | `src/core/toolpack.ts` | Shared tool-pack builder for brain + council (core, http, skill, research, orchestration) |
| **Evidence** | `src/core/evidence.ts` | Shared singleton `EvidenceLedger` for all tools + gate + council |
| **Blackboard** | `src/core/blackboard.ts` | Unified fact/intent state-space + Plan model + tool-call dedup |
| **Approval** | `src/core/approval.ts` | Re-exports council approval as shared gate for both strategies |

### Council Engine (`src/council/`)

Parallel debate: 4 LLM members (strategist, operator, skeptic, analyst) debate what to test, analyze results together. Human steers via HITL approval.

| Module | Location | Purpose |
|--------|----------|---------|
| **Types** | `src/council/types.ts` | `MemberOutput`, `CouncilIntent`, `ImpactLevel`, `CouncilProposal`, `CouncilCritique`, `CouncilReflection`, `DebateCycleResult` |
| **Personas** | `src/council/personas.ts` | LLM persona strings per role + mandatory structured output contract (JSON block) |
| **Factory** | `src/council/factory.ts` | Builds real LLM-backed council agents, `parseStructuredOutput()` extracts typed JSON |
| **Orchestrator** | `src/council/orchestrator.ts` | `debateOnce()` (parallel, one cycle), `runCouncil()` (backward-compat loop) — zero regex |
| **Bus** | `src/council/bus.ts` | `ConversationBus`: append-only log + sliding-window transcript for members |
| **Blackboard** | `src/council/blackboard-shared.ts` | `SharedBlackboard` adapter wrapping core `Blackboard` for council |
| **Evidence Bridge** | `src/council/evidence-bridge.ts` | `bridgeWorkerToolCall()`, `bridgeWorkerEvidence()`, `extractProposedTasks()` — structured extraction |
| **Approval** | `src/council/approval.ts` | `classifyImpact()` reads typed `proposal.impact` field, zero regex. HITL gate for critical. |

**Design principle:** No hardcoded substring detection. Structured typed fields at all seams. All regex removed from council code.

### Intelligence Layer (`src/intelligence/`)

| Module | Location | Purpose | Tests |
|--------|----------|---------|-------|
| **Evidence Gate** | `src/intelligence/evidence-gate.ts` | Anti-hallucination: cross-checks LLM claims against typed evidence | 14 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Msalways/Ultimatrix](https://github.com/Msalways/Ultimatrix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->

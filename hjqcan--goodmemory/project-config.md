---
trigger: always_on
description: > **Are you an agent here to _adopt_ GoodMemory (give yourself or your host
---

# Repository Guidelines

> **Are you an agent here to _adopt_ GoodMemory (give yourself or your host
> durable memory), not to _contribute_ to it?** Stop reading this file — it is
> the contributor guide. Go to [llms.txt](./llms.txt) for a machine-readable
> onboarding decision tree, or the
> [README Quickstart](./README.md#quickstart-codex-or-claude-code-memory). The
> capability descriptor at [.well-known/goodmemory.json](./.well-known/goodmemory.json)
> has install commands, the MCP endpoint, and HTTP endpoints as JSON.
>
> The rest of this document is for agents and humans **working on** the
> GoodMemory codebase.

## Project Structure & Module Organization

Treat this file as a routing layer, not the final authority. Start with
`docs/README.md` for documentation routing and `task-board/00-README.txt` for
execution order. Do not bulk-read `docs/`, `task-board/`, `reports/`, or
`docs/archive/` unless a task explicitly needs historical provenance.

```text
README.md
docs/
├── README.md                                                 # documentation router and archive policy
├── GoodMemory-Current-Status-and-Evidence.md                 # current public surface and canonical evidence
├── GoodMemory-First-Principles-and-Reference-Architecture.md  # canonical design, core beliefs, operating principles
├── GoodMemory-ImplicitMemBench-Full-300-Research-Summary.md    # ImplicitMemBench Full-300 research summary (0.691 claim)
├── GoodMemory-OSS-Architecture-v1.md                          # historical v1 map of domains, packages, and boundaries
├── GoodMemory-PRD.md                                          # product scope and behavior contract
├── GoodMemory-TDD-and-Evaluation-Strategy.md                  # test pyramid, eval design, fixture strategy
├── GoodMemory-v1-Quality-Gate.md                              # historical v1 verification snapshot
├── GoodMemory-v1-Release-Checklist.md                         # historical release readiness baseline
├── GoodMemory-Unified-Self-Evolving-Roadmap.md                # historical roadmap after the v1 core
├── archive/quality-gates/README.md                            # archived phase closure summaries and gate index
├── archive/design-inputs/                                     # superseded drafts, not current truth
├── archive/reference-corpus/                                  # copied research/source material, targeted lookup only
├── GoodMemory-记忆数据分层设计.md                               # layering and storage reference
└── ...

task-board/
├── 00-README.txt                                              # canonical execution order, status markers, working rules
├── 01-phase-0-...txt -> 25-phase-24-...txt                    # phase-level execution plan
└── phase-*/00-README.txt                                      # per-phase breakdown and acceptance criteria

adr/
├── ADR-001-memory-taxonomy.txt
├── ADR-002-public-api.txt
├── ADR-003-runtime-context-controls.txt
├── ADR-004-maintenance-engine.txt
├── ADR-005-scenario-fitted-recall-boundary.txt        # dual-metric recall + scenario-rule admission
├── ADR-006-module-layering-and-shared-contracts.txt   # domain/ contract home, provider ↛ eval
└── ADR-007-python-client-and-docker-distribution.txt   # Python client + Docker distribution

src/
├── index.ts                                                   # package root exports
├── api/                                                       # createGoodMemory surface plus internal runtime/governance wiring
├── ai-sdk/                                                    # AI SDK-facing public exports and contracts
├── domain/                                                    # taxonomy, scope, provenance, core records
├── remember/                                                  # write path: extraction, classification, candidate handling
├── recall/                                                    # retrieval planning, scoring, selection, context assembly
├── answer/                                                     # answer evidence-pack composition and operation guides
├── runtime/                                                   # session-scoped context services and spillover controls
├── maintenance/                                               # decay, dream, consolidation, and maintenance runners
├── verify/                                                    # verification policy for stale or inferred memory
├── storage/                                                   # in-memory, sqlite, postgres, and repository adapters
├── eval/                                                      # eval runners, judge integration, reporting
├── evidence/ evolution/ governance/                           # evidence records, proposal flow, and governance helpers
├── embedding/ provider/                                       # vector write plumbing and provider-backed model adapters
├── host/                                                      # host-facing integration surface and exported contracts
├── language/                                                  # locale-aware extraction and normalization
├── policy/ testing/                                           # policy hooks and shared test helpers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hjqcan/GoodMemory](https://github.com/hjqcan/GoodMemory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

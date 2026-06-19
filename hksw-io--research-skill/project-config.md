---
trigger: always_on
description: >
---


# Multi-Agent Research

Spawn a team of investigators to research a topic from multiple facets in parallel. Inspired by [Anthropic's multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system).

## Arguments

Parse `$ARGUMENTS`:
- `size:N` -> team size override (3-7)
- `depth:X` -> focused (1-2 rounds), standard (2-3), comprehensive (4+)
- Remaining text -> research topic

## Flow

```
CONTEXT_GATHERING -> SCOPE_FRAMING -> PLAN_TO_MEMORY -> INVESTIGATOR_SELECTION -> PARALLEL_INVESTIGATION -> [EVALUATE: more needed?] -> CROSS_POLLINATION -> [EVALUATE: more needed?] -> VERIFICATION -> CONSOLIDATION -> REPORT -> MEMORY_SUGGESTION -> CLEANUP
```

The research loop is adaptive — after each round, the lead agent evaluates whether more investigation is needed rather than committing to a fixed number of rounds upfront.

## Context Gathering

Before framing scope or selecting investigators, the lead agent must build understanding of the project and topic.

### Codebase exploration

If the topic is not about code (e.g., market research, policy questions, product strategy), skip codebase exploration. Proceed directly to high-impact questions.

Use Glob, Grep, and Read to understand the project:
- **Platform detection**: Look for .xcodeproj/.xcworkspace (Apple), .csproj/.sln (Microsoft), AndroidManifest.xml (Android), package.json (Web).
- **Tech stack**: Read project config files, dependency manifests, and top-level source directories.
- **Relevant code**: If the topic references specific features or systems, read the relevant source files.
- **Project docs**: Read CLAUDE.md, README, or architecture docs if they exist.

### High-impact questions

After codebase exploration, evaluate whether there is enough context. If not, ask the user via AskUserQuestion:

- **Research scope**: "Are you looking for a go/no-go assessment, a risk map, or a full implementation plan?"
- **Known constraints**: "What do you already know? What have you already ruled out?"
- **Decision timeline**: "Is this informing an imminent decision or building long-term understanding?"
- **Acceptable depth**: "Should we investigate implementation details or stay at the architectural level?"
- **Prior work**: "Has this been investigated before? What was found?"

Pick the 2-4 most impactful questions. Do not ask generic questions the codebase answers. Do not ask more than 4.

### Context summary

Produce a brief internal context summary (not shown to user) capturing: platform, tech stack, relevant files, constraints, what the user wants to learn. This is included in every investigator's prompt.

## Scope Framing

Score the topic on 4 dimensions (1-3 each):

| Dimension | 1 (Narrow) | 2 (Medium) | 3 (Broad) |
|-----------|-----------|------------|-----------|
| Breadth | Single system/component | 2-3 systems | Cross-cutting concern |
| Depth | Surface-level assessment | Implementation detail needed | Deep technical investigation |
| Uncertainty | Well-understood area | Some unknowns | Mostly unknown territory |
| Stakeholder impact | Single team | 2-3 teams | Organization-wide |

Total = sum (range 4-12).

| Score | Scale | Investigators | Tool call budget/agent |
|-------|-------|--------------|----------------------|
| 4-6 | Focused | 3 | 3-10 calls |
| 7-9 | Standard | 3-5 | 10-15 calls |
| 10-12 | Comprehensive | 5-7 | 15+ calls |

The tool call budget is embedded in each investigator's prompt. Per Anthropic: "agents struggle to judge appropriate effort, so we embedded scaling rules in the prompts."

Override team size with `size:N`.

### Save plan to memory

After scope framing and before spawning investigators, the lead agent saves its investigation plan (topic, facet decomposition, investigator assignments) to persist context. This prevents loss if the context window is truncated during long investigations.

## Investigator Selection

Read [references/perspectives.md](references/perspectives.md) for detailed guidance.

Decompose the research topic into **facets** — distinct aspects that can be investigated independently. Then assign one investigator per facet.

### Facet decomposition

- What technical systems are involved? -> one investigator per major system
- What non-technical dimensions matter? -> (compliance, UX, operations, cost)
- What is the riskiest unknown? -> assign a dedicated investigator

### Structural roles (always present)

1. **Integrator** — maps cross-facet dependencies, identifies cascading effects, produces draft consolidation, and performs the verification step. Domain-labeled (e.g., "Data Pipeline Engineer" who sees how encoding affects every stage).
2. **Scope Sentinel** — monitors whether the investigation is covering the right territory, flags blind spots, asks "what are we not looking at?" Domain-labeled (e.g., "QA Infrastructure Lead" who checks test coverage).

### Facet investigators (1-5 additional)

Each investigator gets clear delegation boundaries:
- **Objective**: What to investigate (concrete, bounded)
- **Output format**: Findings tagged [grounded]/[informed]/[speculative] with confidence levels
- **Tool guidance**: Which tools to prefer, search strategy hints
- **Task boundaries**: What is IN scope and what is explicitly OUT of scope (prevents overlap)

### Codex assignment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hksw-io/research-skill](https://github.com/hksw-io/research-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

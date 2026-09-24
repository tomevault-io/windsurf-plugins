---
trigger: always_on
description: Use GraphFlow first for token-efficient repo context, planning, and orchestration.
---


# GraphFlow Token-First Rule

GraphFlow is a graph-based context and planning service backed by a persistent MCP server. It turns codebases into queryable knowledge graphs, delivering token-efficient compressed context, task planning, and orchestration.

Before broad code exploration, implementation, debugging, review, planning, or architecture questions:

1. Call `graphflow_context` with the user's task/query.
2. Use the returned `summary`, `anchors`, `refillPreview`, and `tokenBudget` as the first context source.
3. Read full files only when:
   - GraphFlow anchors point to that file/symbol,
   - the compressed context is insufficient,
   - or exact edits require the file body.
4. For multi-step or ambiguous work, call `graphflow_plan` before implementation.
5. For graph freshness after project changes, call `graphflow_index`.
6. Report token budget when available:
   - estimated raw tokens,
   - compressed tokens,
   - estimated savings percent,
   - max context token budget.

Do not scan the whole repository, recursively inspect many files, or read large files before trying GraphFlow context.

## Workspace root (`rootDir`)

Pass `rootDir` = the absolute path of the project you are working in. **Never** pass your home directory, AppData, or an unexpanded `${workspaceFolder}` placeholder — GraphFlow refuses unsafe workspace roots and the call fails. If a tool answers `unsafe workspace root`, retry the same call without `rootDir` (the server then uses its configured workspace) or with the project path.

## Chinese / CJK queries (agent must translate)

Code symbols are mostly English. For Chinese user questions:

1. **Proactive:** Before or with `graphflow_context`, translate intent to English **file/class/component names** (e.g. `PoseDetectionPage`, `BattlePage`, `shieldEffect`) and pass `englishQuery`. Avoid generic terms like `exercise` when the user means UI/camera — they often match data/types layers.
2. **Module families (store/slices):** Prefer file stems (`useGameStore companionSlice dailySlice inventorySlice`), not bare domain words like `monster` (often hits `data/monsters` instead of `monsterSlice`).
3. **Reactive:** If preview returns `agentWorkItems` with `query-translate-en` (low `anchorCount`), answer the JSON prompt with your model, then retry preview with `englishQuery`.
4. Keep `query` as the original Chinese text; use `englishQuery` for search terms only.

```typescript
graphflow_context({
  query: "游戏战斗系统怎么实现的",
  englishQuery: "battle combat fight damage scene system",
  rootDir: "/absolute/path/to/project"
})
```

## Tool Inventory (10 MCP Tools)

### Core Context Tools (Highest Frequency)

| Tool | Purpose | Call Frequency |
|------|---------|---------------|
| `graphflow_context` | Preview compressed context (query) or expand anchor (anchorId) | **Highest** - default first step |

### Planning Tools (High Frequency)

| Tool | Purpose | Call Frequency |
|------|---------|---------------|
| `graphflow_plan` | Multi-step task decomposition & DAG (mode='simple' or 'insight') | High - before complex work |
| `graphflow_run` | Plan + context package (bridge mode) | Medium - full task packaging |
| `graphflow_report_outcome` | Report bridge-mode execution outcome back | Medium - close the learning loop |
| `graphflow_insight` | Submit or merge agent insights | Medium - no external LLM API |

### Graph Management Tools (Medium Frequency)

| Tool | Purpose | Call Frequency |
|------|---------|---------------|
| `graphflow_index` | Incremental workspace re-index, single-file, or full rebuild | Medium - after file changes |

### Collaboration & Insights Tools (Low Frequency)

| Tool | Purpose | Call Frequency |
|------|---------|---------------|
| `graphflow_artifact` | Export or import graph artifact | Low - team sharing |
| `graphflow_skill_insights` | Learned skill patterns | Low - leverage prior learning |
| `graphflow_skill_guide` | Skill usage guide for connected agents | Low - onboarding |
| `graphflow_diagnose` | Provider health, graph stats, and token savings | Rare - config issues |

## Standard Workflows

### Workflow 1: Context First (90% of tasks)

**Use when:** Answering code questions, exploring codebase, understanding modules

```
Step 1: graphflow_context(query: "<your question>")
Step 2: Read summary + anchors as primary context
Step 3: Expand specific anchors with graphflow_context(anchorId: "...") when needed
Step 4: Read full files only when exact edits required
Step 5: After answering, graphflow_context({ assistantReply: "<original answer>" }) to fill the pending turn
```

Each `graphflow_context` preview records the user question into the graph (workbench topic if a plan DAG exists, otherwise a dialogue-turn). After answering, call again with `assistantReply` (query optional) so the original answer is stored. Workbench titles are display labels only — do not replace stored messages with an extracted abstract.

### Workflow 2: Plan Before Coding (complex tasks)

**Use when:** Multi-step changes, refactors, features with unclear scope

```
Step 1: graphflow_context(query: "<task>")
Step 2: graphflow_plan(task: "<task description>")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Roarpeng/GraphFlow](https://github.com/Roarpeng/GraphFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

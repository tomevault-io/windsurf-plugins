---
trigger: always_on
description: Use when designing or building AI agent systems, choosing between workflow patterns, diagnosing poor agent decisions, designing tools for agents, setting up evals, or managing context rot in long-running agents.
---


# Agent System Architect

## When This Skill Activates

This skill applies when you encounter ANY of these signals:

- Building or modifying an agent that makes LLM API calls
- Designing tool schemas or MCP servers for agent consumption
- Setting up agent evaluation (evals)
- Debugging agent behavior (wrong tool calls, context issues, poor decisions)
- Choosing between single agent vs multi-agent vs workflow
- Managing long-running agent sessions (context rot, state persistence)
- Implementing RAG for knowledge-heavy agents

**Core principle:** Start with the simplest solution that meets the need. Complexity is a cost, not a feature. Five disciplines compound each other: architecture selection, tool design, context engineering, think tool, and evals. Get one wrong and the others cannot compensate.

---

## First: Assess the Situation

Before recommending anything, answer these questions by examining the codebase:

1. **Lifecycle stage:** Are we starting from scratch (0-to-1), adding features (growing), or debugging (struggling)?
2. **Architecture:** Single agent? Multi-agent? Workflow? What pattern is currently in use?
3. **Tool inventory:** How many tools? How are they designed? What's the token budget for tool definitions?
4. **Context health:** How is context managed? Any signs of context rot (degraded quality over conversation length)?
5. **Eval status:** Do evals exist? What do they cover? Are they saturated?

Based on this assessment, follow the appropriate section below.

---

## Architecture Decision Engine

Think through this step by step. Do not prescribe an architecture without reasoning through each step.

### Step 1: Complexity Assessment

```
If < 5 subtasks AND predictable outcomes --> Lean toward WORKFLOW
If > 5 subtasks OR unpredictable outcomes --> Lean toward AGENT
```

### Step 2: Parallelism Check

```
If subtasks run independently --> Multi-agent (Orchestrator-Workers)
If subtasks are sequential    --> Single agent or Prompt Chain
```

### Step 3: Quality Requirements

```
If output needs iterative refinement AND clear eval criteria --> Evaluator-Optimizer
If output needs refinement BUT no clear criteria            --> Human-in-the-loop
If no refinement needed                                     --> Simpler pattern
```

### Step 4: Input Diversity

```
If different inputs need different handlers AND > 3 categories --> Router
If < 3 categories --> Handle in system prompt with conditional logic
```

### Step 5: Confirm Architecture

| Pattern | When to Use | Complexity | Token Cost |
|---------|------------|:----------:|:----------:|
| Augmented LLM | Single task, single call, predictable | Lowest | $ |
| Prompt Chain | Sequential steps with validation gates | Low | $$ |
| Router | Multiple input types need different handlers | Low | $$ |
| Parallelization | Independent chunks or voting for reliability | Medium | $$ |
| Orchestrator-Workers | Dynamic decomposition, unknown subtask count | High | $$$ |
| Evaluator-Optimizer | Iterative refinement, subjective quality | High | $$$ |

### Real-World Examples

| Use Case | Wrong Choice | Right Choice | Why |
|----------|-------------|--------------|-----|
| Email triage | Autonomous agent | Router | 3 categories, deterministic |
| Code migration | Prompt chain | Orchestrator-Workers | File count unknown, parallelizable |
| Blog post writing | Single LLM call | Evaluator-Optimizer | Quality is subjective, benefits from iteration |
| Customer support | Complex router | Prompt chain + think tool | Sequential policy checks, not routing |
| Data pipeline QA | Autonomous agent | Prompt chain | Steps are fixed and well-defined |

### Performance Characteristics

| Pattern | Latency | Token Cost | Reliability | Debuggability |
|---------|:-------:|:----------:|:-----------:|:-------------:|
| Augmented LLM | Lowest | $ | Highest | Trivial |
| Prompt Chain | Low | $$ | High | Easy (step-by-step) |
| Router | Low | $$ | High | Easy (check classification) |
| Parallel Workers | Low (parallel) | $$ | High | Medium |
| Evaluator-Optimizer | Medium-High | $$$ | Medium | Medium |
| Orchestrator-Workers | High | $$$-$$$$ | Medium | Hard |
| Autonomous Agent | Unpredictable | $$$$ | Lowest | Hardest |

**CRITICAL:** Prefer workflows over autonomous agents for well-defined tasks. Workflows are deterministic and debuggable. Agents are flexible but unpredictable.

See `references/patterns-reference.md` for full implementation with case studies.

---

## Tool Design Clinic

Tools dominate agent context. Poor tools destroy good architectures. When designing or reviewing tools, run this diagnostic.

### Diagnostic 1: Token Budget

Count total tokens across all tool definitions. If > 10K tokens, implement Tool Search Tool (lazy-load). Result: ~85% context reduction.

### Diagnostic 2: Granularity Check

For each tool, ask: "Does this require the agent to make multiple calls to achieve one logical action?"

```
BAD:  list_users() -> get_user_calendar() -> check_availability() -> create_event()
GOOD: schedule_meeting(participants, time, title)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PeterHiroshi/building-agent-systems](https://github.com/PeterHiroshi/building-agent-systems) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

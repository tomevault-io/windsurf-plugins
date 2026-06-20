---
trigger: always_on
description: |
---


# Agent System — Professional Mermaid Documentation Generator

Produce a **complete, deliverable documentation package** from Agent System source code. The output must be accurate enough to be used by engineers who didn't write the code, and professional enough to be presented to technical stakeholders.

---

## Step 0: Pre-Analysis — Read Everything First

Before drawing a single diagram, perform a **full code inventory**. Read ALL provided files. Extract and record:

### 0.1 Component Inventory (fill this in mentally before proceeding)

| Category           | Items found                                               | Source file/line |
| ------------------ | --------------------------------------------------------- | ---------------- |
| **Agents**         | names, roles, which LLM each uses                         | —                |
| **Tools**          | names, signatures, external APIs called                   | —                |
| **State/Schema**   | TypedDict fields, Pydantic models, reducers               | —                |
| **Orchestration**  | graph edges, conditional routing logic, FINISH conditions | —                |
| **Memory**         | checkpointers, vector stores, conversation buffers        | —                |
| **Config**         | model names, temperature, max_tokens, API endpoints       | —                |
| **Async**          | async def, asyncio.gather, parallel branches, Send()      | —                |
| **Error handling** | try/except, retries, fallback agents, max_iterations      | —                |

### 0.2 Architecture Classification

Determine before drawing:
- **Pattern type**: Supervisor/Worker | Pipeline | ReAct Loop | GroupChat | Swarm/Handoff | Custom
- **Concurrency model**: Sequential | Parallel branches | Async event-driven
- **State model**: Shared mutable state | Message-passing | Stateless | External DB
- **Memory model**: In-memory | Persisted (checkpointer) | Vector retrieval | None
- **Framework**: LangGraph | AutoGen | CrewAI | Pydantic AI | DSPy | Semantic Kernel | LlamaIndex | Custom | Mixed

> **⚠️ MANDATORY:** Before proceeding, consult `references/patterns.md` to match the source code against each framework's **identifying markers**. Use the matched framework's canonical template as your diagram starting point. Do NOT guess the framework — match by code markers. For ambiguous or mixed-framework projects, identify EACH framework separately.

If framework is ambiguous, look for:
- Graph-based: `add_node`, `add_edge`, `StateGraph`, node functions returning state dicts
- Message-passing: `initiate_chat`, `GroupChat`, reply functions, termination conditions
- Declarative task: `Agent(role=...)`, `Task(description=...)`, `Crew`, `kickoff()`
- Functional/DSP: `@dspy.module`, `Predict`, `ChainOfThought`, `TypedPredictor`

### 0.3 Accuracy Rules — Always Follow

**EXPLICIT** (solid lines in diagrams): relationships directly expressed in code — `add_edge`, `goto`, function calls, class inheritance.

**INFERRED** (dashed lines + footnote): relationships implied by naming, structure, or LLM behavior — e.g., "this agent likely calls the LLM based on its base class."

**UNKNOWN** (node name ends in `?`, dashed border): components referenced but not in provided files.

Never draw a solid arrow for something you inferred. Mark every inference explicitly.

### 0.4 Mermaid Syntax Guard — Mandatory Pre-flight

Before writing any Mermaid code, internalize these critical rules (full reference: `references/syntax-guard.md`):

1. **`%%{init}` must be line 1** of every Mermaid block — no blank lines before it, use single quotes inside JSON
2. **Labels with special characters** → always wrap in `["..."]`: `A["function(arg)"]` not `A[function(arg)]`
3. **Subgraph IDs** → alphanumeric + underscore only: `subgraph AgentLayer` not `subgraph Agent Layer`
4. **Subgraph titles** → use quoted bracket syntax: `subgraph ID["Title With Spaces"]`
5. **Edge labels with spaces** → always quote inside pipes: `-->|"label text"| B`
6. **`classDef` before use** → define ALL `classDef` lines at the top of the diagram, before any node references
7. **No HTML in labels** → use `<br/>` for line breaks in flowcharts, `\n` in sequence participant aliases
8. **30-node limit** → if >30 nodes in a single diagram, split into C4 layers (see Special Cases)
9. **Bracket matching** → every `[`, `(`, `{`, `"` must have a matching closer; every `subgraph`/`loop`/`alt`/`par` must have `end`
10. **Activation balancing** → in sequence diagrams, every `+` (activate) must have a matching `-` (deactivate)

> **After generating each diagram, mentally validate against `references/syntax-guard.md` §8 Quick Reference Checklist before outputting. If a diagram is complex (>15 nodes), run the Self-Repair Protocol from §6 preemptively.**

### 0.5 Scope & Degradation Strategy

Not all projects warrant all 8 diagrams. Determine scope before drawing:

| Condition | Action |
|-----------|--------|
| Code < 50 lines, 1 agent, no tools | Generate Diagram 1 only + Architecture Summary |
| Code < 150 lines, 2-3 agents | Generate Diagrams 1 + 2 + 3 (mandatory set only) |
| Standard project (3-15 components) | Generate Diagrams 1-3 (mandatory) + all applicable optional diagrams |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WYXNICK/agent-mermaid-skill](https://github.com/WYXNICK/agent-mermaid-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->

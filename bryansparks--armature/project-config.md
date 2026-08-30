---
trigger: always_on
description: Armature is a DAG-based multi-agent workflow engine for Python. Developers describe agentic
---

# Armature — AI Coding Guide (Codex / OpenAI CLI)

Armature is a DAG-based multi-agent workflow engine for Python. Developers describe agentic
teams as YAML spec files; the engine resolves execution order, manages context flow between
stages, calls LLMs, and handles retries and fan-out automatically.

## The Primary Path

**In Codex (or any AI coding tool), this is the best way to create a workflow:**

1. Describe the agentic team you want to build in natural language
2. Ask Codex to generate a complete, documented YAML spec
3. Validate: `armature validate my_workflow.yml`
4. Run: `armature run my_workflow.yml --input key=value`

**Working reference:** `docs/ARMATURE-SPEC-REF.md` — all fields and valid values on one page. Load this as context when generating a spec.
**Full documentation:** `docs/USER-GUIDE.md` — consult for advanced features (fan-out, memory, safety rules, lifecycle hooks, deliberative teams).

---

## Core Mental Model

### DAG + cumulative context
Every stage declares `depends_on`. The engine runs them in topological order. Each completed
stage stores its result in a shared context dict under `stage_id`. Every downstream stage
automatically sees all prior outputs — **no explicit data wiring is needed**.

```
researcher ──► analyst ──► judge ──► writer
                  ↑
       (sees researcher's output automatically)
```

### Stage types
| Field | What it does |
|-------|-------------|
| `role:` | LLM stage — has a system prompt, produces text or JSON |
| `tool_call:` | Direct tool invocation — no LLM, deterministic |
| `gate: human` | Pauses execution for human approval |
| `adapter:` | Runs a shell command or Python function |
| `subagent_spec:` | Spawns a child workflow as a nested execution |

### Model tiers
Define named slots (`tiny`/`small`/`medium`/`large`/`frontier`). Stages reference tier names,
not model names directly. Swap all models globally by editing the `model_tiers` block.

### Output modes
- `output_mode: text` → stored as `{"content": "..."}` — use for freeform responses
- `output_mode: guided_json` + `output_schema:` → strongly typed dict; auto-escalates to next
  tier on parse failure — use whenever downstream stages depend on specific fields

---

## Generating a Spec — Checklist

When generating workflow YAML, produce all of these sections in order:

**1. Header**
```yaml
name: my_workflow
version: "1.0"
description: "One sentence describing what this workflow does."
mission: >
  Optional paragraph ALL LLM stages inherit as background context.
  Use for workflow-level invariants: tone, constraints, domain knowledge.
```

**2. Contracts**
```yaml
contracts:
  inputs:
    - name: topic          # every runtime input must be declared here
    - name: focus          # optional inputs too — document them all
  max_iterations: 40
  max_llm_calls: 200
  timeout_hours: 1.0
```

**3. Model tiers**
```yaml
model_tiers:
  small:
    provider: openai
    model: gpt-4o-mini
    temperature: 0.2
    max_tokens: 2048
  large:
    provider: openai
    model: gpt-4o
    temperature: 0.3
    max_tokens: 16000

role_type_defaults:
  worker: small
  orchestrator: large
  judge: large
  researcher: large
```

**4. Tools (if workflow uses custom tools)**
```yaml
tools:
  - module: my_package.tools.web     # Python module path; must define register(registry)
```

**5. Stages** — see patterns below

**6. Post-run self-analyst** — add for any non-trivial workflow:
```yaml
- id: self_analyst
  post_run: true
  fail_as_value: true
  depends_on: []
  signature:
    input:
      # List the key output stages — NEVER use _transcript in fan-out workflows
      topic: Research topic
      synthesizer: The final synthesis output
      judge: Quality assessment
  role:
    name: Director
    type: judge
    description: |
      Review this completed run. Identify quality issues and suggest improvements
      to stage prompts, model tier assignments, or workflow structure.
```

---

## Common Patterns

### Linear pipeline (planner → workers → judge)
```yaml
stages:
  - id: planner
    role: {name: Planner, type: orchestrator, description: "Plan the approach for: {{ topic }}"}
    output_mode: guided_json
    output_schema:
      type: object
      required: [steps]
      properties:
        steps: {type: array, items: {type: string}}
    depends_on: []

  - id: executor
    role: {name: Executor, type: worker, description: "Execute: {{ planner.steps }}"}
    output_mode: text
    depends_on: [planner]

  - id: judge
    role: {name: Judge, type: judge, description: "Assess quality of: {{ executor.content }}"}
    output_mode: guided_json
    output_schema:
      type: object
      required: [accept, confidence, issues]
      properties:
        accept: {type: boolean}
        confidence: {type: number}
        issues: {type: array, items: {type: string}}
    on_fail:
      loop: {stage: judge, max: 2}
    depends_on: [executor]
```

### Fan-out research pipeline (parallel search + synthesis)
```yaml
  - id: plan_searches
    role: {name: SearchPlanner, type: worker, description: "Generate search queries for {{ topic }}"}
    output_mode: guided_json
    output_schema:
      type: object
      required: [queries]
      properties:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bryansparks/armature](https://github.com/bryansparks/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->

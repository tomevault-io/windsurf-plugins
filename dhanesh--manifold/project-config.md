---
trigger: always_on
description: Specialized agents for constraint-first development. Each agent handles a specific phase of the Manifold workflow.
---

# Manifold Agents

Specialized agents for constraint-first development. Each agent handles a specific phase of the Manifold workflow.

## Agent Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                        MANIFOLD ORCHESTRATOR                          │
│  Routes to specialized agents based on current phase and task type    │
└──────────────────────────────────────────────────────────────────────┘
        │
        ├─────────────────────────────────────────────────────────────┐
        │                                                             │
        ▼                                                             ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   CONSTRAINT    │  │    TENSION      │  │     ANCHOR      │  │   GENERATION    │
│   DISCOVERY     │  │    ANALYSIS     │  │     AGENT       │  │     AGENT       │
│     AGENT       │  │      AGENT      │  │                 │  │                 │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│ Phase: m1       │  │ Phase: m2       │  │ Phase: m3       │  │ Phase: m4       │
│ Elicits across  │  │ Finds conflicts │  │ Backward        │  │ Creates code,   │
│ 5 categories    │  │ Suggests fixes  │  │ reasoning       │  │ tests, docs,    │
└─────────────────┘  └─────────────────┘  └─────────────────┘  │ runbooks, alerts│
        │                     │                   │            └─────────────────┘
        │                     │                   │                    │
        ▼                     ▼                   ▼                    ▼
┌─────────────────┐                                            ┌─────────────────┐
│  VERIFICATION   │◀───────────────────────────────────────────│   INTEGRATION   │
│     AGENT       │                                            │     AGENT       │
├─────────────────┤                                            ├─────────────────┤
│ Phase: m5       │                                            │ Phase: m6       │
│ Validates all   │                                            │ Wires artifacts │
│ artifacts       │                                            │ together        │
└─────────────────┘                                            └─────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      PARALLEL EXECUTION AGENT                        │
│  Manages worktree-based parallelism for independent tasks            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1. Constraint Discovery Agent

**Command:** `/manifold:m1-constrain <feature>`

**Purpose:** Elicit and document ALL constraints across 5 categories before any implementation begins.

### Activation Triggers

- Phase is `INITIALIZED`
- User invokes `/manifold:m1-constrain`
- New feature needs constraints identified

### Behavior

1. Load existing manifold from `.manifold/<feature>.json` + `.manifold/<feature>.md`
2. For each category, ask probing questions (see Interview Protocol)
3. Classify responses into constraint types (`invariant`, `goal`, `boundary`)
4. Assign IDs (B1, T1, U1, S1, O1, etc.)
5. Update manifold JSON + MD files
6. Set phase to `CONSTRAINED`

### Interview Protocol

```yaml
business:
  questions:
    - "What is the revenue/cost impact of this feature?"
    - "Are there compliance or legal requirements?"
    - "Who are the stakeholders and what do they need?"
    - "What happens if this feature fails?"
  invariant_signals: ["must", "cannot", "never", "always", "required by law"]
  goal_signals: ["should", "ideally", "target", "aim for"]
  boundary_signals: ["maximum", "minimum", "within", "no more than"]

technical:
  questions:
    - "What are the performance requirements?"
    - "What systems does this integrate with?"
    - "What data consistency guarantees are needed?"
    - "What is the expected load/scale?"
  invariant_signals: ["ACID", "consistency", "must not lose"]
  boundary_signals: ["SLA", "latency", "throughput", "timeout"]

user_experience:
  questions:
    - "What response times are acceptable?"
    - "How should errors be communicated?"
    - "Are there accessibility requirements?"
    - "What is the user's mental model?"
  invariant_signals: ["WCAG", "must be accessible"]
  goal_signals: ["intuitive", "easy", "clear"]

security:
  questions:
    - "What data needs protection?"
    - "What authentication/authorization is required?"
    - "What needs to be audited?"
    - "What are the threat vectors?"
  invariant_signals: ["PII", "encrypt", "audit", "compliance"]
  boundary_signals: ["authentication required", "role-based"]

operational:
  questions:
    - "What needs monitoring?"
    - "What are the SLA requirements?"
    - "What is the incident response process?"
    - "How will this be deployed/rolled back?"
  invariant_signals: ["alerting", "logging required"]
  goal_signals: ["observable", "debuggable"]
```

### Output Format

```
CONSTRAINT DISCOVERY: <feature>

CONSTRAINTS DISCOVERED:

Business:
- B1: <statement> (invariant)
- B2: <statement> (goal)

Technical:
- T1: <statement> (boundary)
...


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dhanesh/manifold](https://github.com/dhanesh/manifold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

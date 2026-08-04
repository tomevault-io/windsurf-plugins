---
trigger: always_on
description: Create specialized agents with type subscriptions, custom evaluation, and lifecycle hooks
---


# Agents

Agents are the **autonomous workers** in Flock. They consume data from the [blackboard](blackboard.md), transform it, and publish results back—all without knowing about each other.

**Think of agents like skilled craftspeople in a workshop:** each specialist watches the shared workspace (blackboard) for materials they can work with, processes them independently, and places finished work back on the table for others.

---

## What is an Agent?

An agent is defined by three things:

1. **What it consumes** - The input data types it watches for
2. **What it publishes** - The output data types it produces
3. **How it transforms data** - The logic (usually LLM-powered) that converts input to output

**Key principle:** Agents are **loosely coupled**. They don't call each other directly—they just subscribe to data types on the blackboard.

---

## Your First Agent (60 Seconds)

Here's a complete working example:

```python
from pydantic import BaseModel, Field
from flock import Flock, flock_type

# 1. Define input and output types
@flock_type
class PizzaIdea(BaseModel):
    description: str

@flock_type
class Pizza(BaseModel):
    ingredients: list[str]
    size: str
    crust_type: str
    instructions: list[str]

# 2. Create orchestrator and agent
flock = Flock("openai/gpt-4.1")

pizza_master = (
    flock.agent("pizza_master")
    .consumes(PizzaIdea)
    .publishes(Pizza)
)

# 3. Use it
await flock.publish(PizzaIdea(description="truffle pizza"))
await flock.run_until_idle()
```

**What just happened:**
- ✅ **No prompts written** - The `Pizza` schema defines the output structure
- ✅ **Type-safe execution** - Pydantic validates the LLM output
- ✅ **Self-documenting** - The code shows exactly what the agent does
- ✅ **Future-proof** - Works with GPT-4, GPT-5, Claude, any model

[**👉 Try this example**](https://github.com/whiteducksoftware/flock/blob/main/examples/01-the-declarative-way/01_declarative_pizza.py)

---

## Agent Builder API

### Basic Agent Creation

```python
agent = flock.agent("agent_name")
```

Every agent needs a unique name within the flock instance.

### Adding a Description

```python
agent = (
    flock.agent("analyzer")
    .description("Analyzes customer sentiment from reviews")
)
```

**Why add descriptions?**
- Improves multi-agent coordination (agents understand each other's roles)
- Helps debugging (clear purpose in traces)
- Self-documents your system

### Declaring Consumption

```python
agent = flock.agent("analyzer").consumes(CustomerReview)
```

**Consume multiple types:**
```python
# Agent waits for BOTH types before executing
diagnostician = (
    flock.agent("diagnostician")
    .consumes(XRayAnalysis, LabResults)
    .publishes(Diagnosis)
)
```

**How it works:**
- Agent subscribes to specific data types
- Automatically triggered when matching artifact published to blackboard
- Can wait for multiple inputs (automatic dependency resolution)

### Declaring Publication

```python
agent = (
    flock.agent("reviewer")
    .consumes(CodeSubmission)
    .publishes(CodeReview)
)
```

**Publish multiple types:**
```python
# Agent can produce different outputs
analyzer = (
    flock.agent("analyzer")
    .consumes(BugReport)
    .publishes(BugAnalysis, SecurityAlert)  # Produces both types
)
```

### Complete Agent Definition

```python
code_reviewer = (
    flock.agent("code_reviewer")
    .description("Reviews code for bugs and security issues")
    .consumes(CodeSubmission)
    .publishes(CodeReview)
)
```

---

## Logic Operations (AND/OR Gates)

Flock provides intuitive syntax for coordinating multiple input types through **AND gates**, **OR gates**, and **count-based logic**. This enables powerful coordination patterns without manual wiring.

### AND Gates: Wait for ALL types

**Use `.consumes(A, B)` to wait for multiple types before triggering:**

```python
# Diagnostician waits for BOTH inputs
diagnostician = (
    flock.agent("diagnostician")
    .consumes(XRayAnalysis, LabResults)  # AND gate: waits for both
    .publishes(Diagnosis)
)
```

**How it works:**
- Agent collects artifacts as they arrive
- Triggers only when **ALL required types** are present
- Order-independent: `XRay → Lab` or `Lab → XRay` both work
- After triggering, the waiting pool clears for next cycle

**Example timeline:**
```
Time 0: XRayAnalysis published → diagnostician waits...
Time 1: LabResults published   → diagnostician triggers! ✅
Time 2: Agent executes with BOTH artifacts
```

**Use cases:**
- Multi-modal fusion (images + text + metadata)
- Parallel analysis aggregation (multiple perspectives → final decision)
- Dependency coordination (wait for prerequisites)

### OR Gates: Trigger on ANY type

**Use chained `.consumes()` to trigger on multiple types independently:**

```python
# Alert handler triggers on EITHER alert type
alert_handler = (
    flock.agent("alert_handler")
    .consumes(SystemAlert)      # OR
    .consumes(UserAlert)        # OR
    .consumes(SecurityAlert)    # OR
    .publishes(AlertResponse)
)
```

**How it works:**
- Each `.consumes()` creates a **separate subscription**
- Agent triggers independently for each type
- Single artifact per trigger (not accumulated)

**Example timeline:**
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whiteducksoftware/flock](https://github.com/whiteducksoftware/flock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

---
trigger: always_on
description: A Python library for evaluating text outputs against weighted criteria using LLM-as-a-judge.
---

# Rubric Package

A Python library for evaluating text outputs against weighted criteria using LLM-as-a-judge.

## Package Structure

```
src/rubric/
├── __init__.py              # Public exports
├── rubric.py                # Core Rubric class
├── types.py                 # Type definitions (Criterion, protocols)
├── utils.py                 # Utility functions (default generators)
└── autograders/
    ├── __init__.py          # Autograder exports
    ├── base.py              # Abstract Autograder base class
    ├── schemas.py           # Pydantic output schemas
    ├── per_criterion_grader.py         # Parallel per-criterion evaluation
    ├── per_criterion_one_shot_grader.py # Single-call all-criteria evaluation
    └── rubric_as_judge_grader.py       # Holistic scoring
```

## Core Types

### `Criterion`
A single evaluation criterion with weight and requirement:
```python
class Criterion(BaseModel):
    weight: float      # Positive for desired traits, negative for errors
    requirement: str   # What to evaluate
```

### `CriterionReport`
A criterion with its evaluation result:
```python
class CriterionReport(Criterion):
    verdict: Literal["MET", "UNMET"]
    reason: str
```

### `EvaluationReport`
Final grading result:
```python
class EvaluationReport(BaseModel):
    score: float                              # Normalized 0-1 (or raw if normalize=False)
    raw_score: float | None                   # Always weighted-sum semantics (consistent across all graders)
    llm_raw_score: float | None               # Original LLM output before conversion
    report: list[CriterionReport] | None      # Per-criterion details (if available)
```

**Score Field Semantics:**
- `raw_score`: Always uses **weighted-sum semantics** regardless of grader type. This ensures training pipelines can use `raw_score` consistently without knowing which grader was used.
- `llm_raw_score`: The **original value** from the LLM before any conversion:
  - `PerCriterionGrader` / `PerCriterionOneShotGrader`: Same as `raw_score` (weighted sum)
  - `RubricAsJudgeGrader`: The 0-100 holistic score from the LLM (useful for debugging)

**Error Handling:**
- Validation happens at generation time via Pydantic models
- If your `generate_fn` returns invalid data, Pydantic will raise a `ValidationError`
- Users control retry logic in their `generate_fn` based on their LLM client's capabilities

## Pydantic Output Schemas

All autograders require typed `generate_fn` implementations that return validated Pydantic models. These schemas ensure strict type safety and enable constrained decoding.

### `PerCriterionOutput`
Used by `PerCriterionGrader` for single-criterion evaluation:

```python
from typing import Literal
from pydantic import BaseModel

class PerCriterionOutput(BaseModel):
    criterion_status: Literal["MET", "UNMET"]
    explanation: str
```

### `OneShotOutput`
Used by `PerCriterionOneShotGrader` for batch evaluation:

```python
from typing import Literal
from pydantic import BaseModel, Field

class CriterionEvaluation(BaseModel):
    criterion_idx: int  # 0-based index
    criterion_status: Literal["MET", "UNMET"]
    explanation: str

class OneShotOutput(BaseModel):
    criteria_evaluations: list[CriterionEvaluation] = Field(min_length=1)
```

### `RubricAsJudgeOutput`
Used by `RubricAsJudgeGrader` for holistic scoring:

```python
from pydantic import BaseModel

class RubricAsJudgeOutput(BaseModel):
    overall_score: float  # 0-100 scale
    explanation: str      # Brief explanation of the score
```

### Accessing Schemas for Constrained Decoding

All schemas expose `.model_json_schema()` for constrained decoding:

```python
from rubric import PerCriterionOutput

# Get JSON schema for your LLM client
schema = PerCriterionOutput.model_json_schema()

# Example with OpenAI structured outputs
response = await openai_client.chat.completions.create(
    model="gpt-4",
    messages=[...],
    response_format={"type": "json_schema", "json_schema": {
        "name": "criterion_output",
        "schema": schema
    }}
)

# Parse response into validated Pydantic model
output = PerCriterionOutput.model_validate_json(response.choices[0].message.content)
```

## Typed GenerateFn Protocols

Each autograder requires a specific typed `generate_fn` protocol that returns a validated Pydantic model:

### `PerCriterionGenerateFn`
For `PerCriterionGrader`:
```python
from rubric import PerCriterionGenerateFn, PerCriterionOutput

async def my_generate_fn(
    system_prompt: str,
    user_prompt: str,
    **kwargs
) -> PerCriterionOutput:
    # Your LLM call here
    return PerCriterionOutput(
        criterion_status="MET",
        explanation="The criterion is satisfied."
    )
```

### `OneShotGenerateFn`
For `PerCriterionOneShotGrader`:
```python
from rubric import OneShotGenerateFn, OneShotOutput, CriterionEvaluation

async def my_generate_fn(
    system_prompt: str,
    user_prompt: str,
    **kwargs
) -> OneShotOutput:
    # Your LLM call here
    return OneShotOutput(
        criteria_evaluations=[
            CriterionEvaluation(
                criterion_idx=0,
                criterion_status="MET",
                explanation="First criterion satisfied"
            ),
            # ... more evaluations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paper-instruments/rubric](https://github.com/paper-instruments/rubric) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->

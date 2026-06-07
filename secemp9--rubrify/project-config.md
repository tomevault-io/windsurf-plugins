---
trigger: always_on
description: This file gives AI coding agents the context needed to work effectively in this codebase. For full user-facing documentation, see README.md.
---

# AGENTS.md -- AI Coding Agent Guide for rubrify

This file gives AI coding agents the context needed to work effectively in this codebase. For full user-facing documentation, see README.md.

---

## Project Overview

rubrify is a rubric compiler and judge engine for LLM evaluation. You define structured evaluation rubrics as typed Python objects (the IR layer), compile them into immutable bundles (the compiler), render them as XML prompts (the codecs), and execute criterion-by-criterion LLM-based judgments against text responses (the engine). An optional evolution module uses GEPA to iteratively optimize rubric text components against human-annotated datasets. All LLM access goes through the `harn_ai` library; all models are Pydantic v2 with `extra="forbid"`.

---

## Tech Stack and Constraints

- **Python >= 3.12** (required by pyproject.toml).
- **Pydantic v2 (>= 2.10)** with `SchemaModel` from `harn_ai.types`. SchemaModel sets `extra="forbid"` by default, meaning any unknown field on any model raises `ValidationError`.
- **harn_ai** for LLM access (multi-provider: OpenAI, Anthropic, DeepSeek, Google, local proxies). Provides `Model`, `Context`, `SimpleStreamOptions`, `Tool`, `UserMessage`, `complete_simple()`, `parse_json_with_repair()`, and `get_env_api_key()`.
- **harn_agent** for agent primitives (dependency, but not heavily used in rubrify's own code).
- **xml.etree.ElementTree** for XML construction and serialization. No XML parsing of untrusted input occurs in this codebase (the XML codec only constructs and serializes documents, never parses them).
- **gepa >= 0.1.0** (optional, for `rubrify[evolve]`). Provides `GEPAAdapter`, `GEPAEngine`, `ReflectiveMutationProposer`, `ParetoCandidateSelector`, `RoundRobinReflectionComponentSelector`, `EpochShuffledBatchSampler`, `MaxMetricCallsStopper`, `AcceptanceCriterion`, `EvaluationBatch`, `GEPAResult`.
- **asyncio throughout the engine**. `execute_criterion()`, `run_judge_loop()`, and `Judge.evaluate()` are all async. The evolution module bridges async-to-sync via `evolve/async_bridge.py`'s `run_async()` utility, which uses `asyncio.run()` when no event loop is running, or dispatches to a `ThreadPoolExecutor` when already inside an async context (e.g. Jupyter, async test harness).
- **Build system**: Hatchling (`hatchling.build`), with `src/rubrify` layout.
- **Workspace sources**: `harn-ai` and `harn-agent` are workspace-local; `gepa` is a path dependency (`../../../gepa`).

---

## Module Map

### `src/rubrify/__init__.py`
Public API surface. Re-exports all core types from ir/, compiler/, codecs/, and engine/. Conditionally imports evolve/ (guarded by `ImportError` for missing gepa).

### ir/ -- Intermediate Representation (Typed Core)

| File | Description | Key Exports |
|---|---|---|
| `ir/__init__.py` | Star-imports from all IR submodules. | -- |
| `ir/types.py` | Scale types (BinaryScale, OrdinalScale, NominalScale, NumericScale), ScaleAnchor, EvidenceSpec, Criterion, CriterionGroup, Disqualifier, Definition, AdviceRule, CalibrationExample, PatternEntry, RubricMeta, Rubric. The Scale union is `Annotated[..., Field(discriminator="kind")]`. Rubric has model validators for unique criterion IDs, valid group refs, and valid disqualifier refs. | `Scale`, `Criterion`, `Rubric`, all scale types |
| `ir/roles.py` | RoleSpec (judge persona with authority, obligations, constraints), SurfacePolicy (codec selection, criterion_focus, decision_thresholds, execution_strategy). `execution_strategy` on SurfacePolicy controls how criteria are dispatched to LLM calls: `"holistic"`, `"grouped"`, or `"per_criterion"` (default). | `RoleSpec`, `SurfacePolicy` |
| `ir/constraints.py` | SurfaceProjection (one codec-specific representation), ConstraintBinding (triple-layer alignment: criterion <-> surface <-> output), AuthorityBlock (instruction vs data marking), OutputConstraint discriminated union of typed variants (PrefixSuffixConstraint, WordCountConstraint, CharLimitConstraint, ItemCountConstraint, TokenConstraint), each with a `check()` method and a `scope` field (`"call"`, `"criterion"`, or `"judgment"`) controlling when the constraint is checked. | `ConstraintBinding`, `SurfaceProjection`, `AuthorityBlock`, `OutputConstraint`, `PrefixSuffixConstraint`, `WordCountConstraint`, `CharLimitConstraint`, `ItemCountConstraint`, `TokenConstraint` |
| `ir/bundle.py` | RubricBundle (frozen Pydantic model, `model_config = ConfigDict(extra="forbid", frozen=True)`). `lock_bundle()` compiles regex patterns from PatternEntry and Disqualifier patterns, fails loudly on invalid regex. | `RubricBundle`, `lock_bundle` |

### compiler/ -- Rubric to RubricBundle Transformation

| File | Description | Key Exports |
|---|---|---|
| `compiler/__init__.py` | Star-imports from compiler.py. | -- |
| `compiler/compiler.py` | `compile_rubric(rubric, *, policy, output_constraints) -> CompilationResult`. Synchronous, pure function. Runs: bind -> authority_blocks -> lock -> audit. `CompilationResult` is a dataclass with `.bundle`, `.issues`, and `.ok` property. | `compile_rubric`, `CompilationResult` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [secemp9/rubrify](https://github.com/secemp9/rubrify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->

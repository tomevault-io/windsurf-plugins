---
trigger: always_on
description: This document provides context, patterns, and guidelines for AI coding assistants working in this repository. For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).
---

# AGENTS.md

This document provides context, patterns, and guidelines for AI coding assistants working in this repository. For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).

## Working with the Community

When helping someone contribute, you are a guide — not a gatekeeper, not a substitute author. The contribution is theirs; help them make it good and learn along the way. The standard for what makes a good contribution lives in [CONTRIBUTING.md](./CONTRIBUTING.md); this is about the people.

- **Point people to the community.** Real questions and design discussion belong with people — the [Discord](https://discord.gg/strands) and [GitHub Discussions](https://github.com/strands-agents/evals/discussions).
- **Assume good faith.** Most contributors are learning; meet them where they are. Good first issues are for bringing newcomers in, not just tickets to close.
- **Talk with contributors, not at them.** Warm, plain, concise. One question at a time, no walls of text, never patronizing. Explain the *why* so it teaches rather than dictates.

## Product Overview

**strands-agents-evals** is an open-source evaluation framework for AI agents and LLM applications. It is part of the Strands Agents ecosystem and is built directly on the Strands Agents SDK.

**Core Features:**
- LLM-as-a-Judge evaluators with built-in rubrics (correctness, faithfulness, helpfulness, etc.)
- Multimodal evaluators (image-to-text tasks)
- Deterministic evaluators (output/trajectory/environment-state)
- Trace-based evaluation over OpenTelemetry sessions from CloudWatch, Langfuse, OpenSearch, LangChain, and Strands in-memory
- Multi-turn conversation simulators (ActorSimulator / UserSimulator / ToolSimulator)
- Failure detection and root-cause analysis (`detectors` module)
- Chaos testing — deterministic tool failure / response corruption (`chaos` module)
- Red-team / adversarial evaluation with strategy library (`experimental.redteam`)
- Automated experiment / test-case generation
- Async parallel execution and per-case result caching (`EvaluationDataStore` / `LocalFileTaskResultStore`)
- Experiment management with JSON serialization

## Relationship to the Strands Agents SDK

strands-evals depends on **`strands-agents`** (the Python SDK) for all LLM interactions. Any change in this repo that touches model calls, tools, sessions, traces, hooks, or structured output should be interpreted against the SDK's public API.

- SDK source (external, not in this repo): [`strands-agents/sdk-python`](https://github.com/strands-agents/sdk-python) under `src/strands/`
- Key SDK surfaces actually imported by this repo (grep-verified):
  - `strands.Agent`, called as `Agent(model=..., system_prompt=..., callback_handler=None)` and invoked via `agent(prompt, structured_output_model=PydanticModel)`
  - `strands.models.model.Model` and provider classes
  - `strands.agent.agent_result.AgentResult`, `strands.agent.conversation_manager.SlidingWindowConversationManager`
  - `strands.tools.decorator` (`DecoratedFunctionTool`, `FunctionToolMetadata`) and the `@tool` decorator from `strands`
  - `strands.types.content.Message`
  - `strands.types.exceptions` (`EventLoopException`, `ModelThrottledException`)
- **Not used as SDK imports (do not add to the list without verifying a real import):**
  - `strands.types.traces` — this repo defines its own `strands_evals.types.trace` for session/span modeling; the SDK's trace types are not imported.
  - `strands.telemetry` — the evals repo has its own `strands_evals.telemetry` module. The only reference to `strands.telemetry` is the literal string `"strands.telemetry.tracer"` used as an OpenTelemetry scope name in `mappers/constants.py`.

**Review guidance:** When reviewing a PR in this repo, if the change uses any SDK feature (agents, tools, models, streaming, structured output, sessions, hooks, telemetry), verify the usage matches the current SDK public API. If the SDK source is available locally (e.g., via a sibling checkout), scan it; otherwise rely on the imported symbols and docstrings. Flag usages that depend on private SDK internals (anything under `_*` modules) or deprecated experimental surfaces.

## Directory Structure

```
strands-evals/
│
├── src/strands_evals/                        # Main package source code
│   ├── __init__.py                           # Public API exports
│   ├── case.py                               # Case[InputT, OutputT] test scenario
│   ├── experiment.py                         # Experiment orchestration + run_evaluations
│   ├── eval_task_handler.py                  # EvalTaskHandler, TracedHandler, @eval_task
│   ├── evaluation_data_store.py              # EvaluationDataStore
│   ├── local_file_task_result_store.py       # LocalFileTaskResultStore
│   ├── _async.py                             # Async helpers
│   ├── utils.py                              # Shared utilities
│   │
│   ├── evaluators/                           # Evaluator implementations
│   │   ├── evaluator.py                      # Base Evaluator[InputT, OutputT]
│   │   ├── coherence_evaluator.py
│   │   ├── conciseness_evaluator.py
│   │   ├── correctness_evaluator.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-agents/evals](https://github.com/strands-agents/evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

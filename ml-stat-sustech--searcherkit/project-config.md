---
trigger: always_on
description: - Do not use broad exception handlers such as `except Exception`.
---

# AGENTS.md

## Additional Rules
- Do not use broad exception handlers such as `except Exception`.
  Catch explicit, concrete exception types instead.
- Do not add compatibility re-export layers by default. After moving code, update
  imports and config targets to the real module paths, then search for stale
  paths.
- After changing config, CLI, recipes, or plugin entry points, run at least a
  compose/help/import check. Do not stop at editing files.
- When running project commands, prefer an already existing/active virtual
  environment. Use `uv run` only when no usable environment can be found.
- For network-related commands or tests, use `uv run` even when a virtual
  environment exists. This includes real network calls, localhost services,
  proxies, and HTTP mocking/interception tools such as `respx` or mock
  transports.
- Always check `great-docs.yml` in any code changes for documentation configuration and consistency.

## Documentation Rules
- Use repository-relative links for links between project documentation files.
  Do not replace internal documentation links with GitHub, deployed-site, or
  machine-local absolute URLs.
- Prefix ordered documentation page filenames with a two-digit sequence and a
  hyphen, for example `01-search-files.qmd` and `02-search-web.qmd`. Keep the
  sequence consistent with the intended sidebar and reading order.
- Keep `index.qmd` as the unnumbered directory entry-point exception. Number
  the other pages in that directory when their relative order matters.
- After renaming a documentation page, update every relative link and search
  the repository for the stale filename before building the docs.

## Testing Rules
- New tests should cover three concerns when they apply: config initialization,
  functional behavior, and retry behavior.
- Config initialization means the same test flow should exercise construction
  through normal non-config arguments and through the config/factory path. For
  example, test the object once with direct constructor parameters and once with
  `config=...` or the module factory that consumes that config.
- Functional tests are the normal behavior checks for the object under test:
  inputs, outputs, request payloads, parsed responses, state changes, and error
  surfaces that are part of the object contract.
- Retry tests are additional functional tests for objects that can encounter
  recoverable errors such as timeouts, transient provider errors, or temporary
  source failures. When such errors exist, include both scenarios: the first
  `n-1` attempts fail and the `n`th attempt succeeds, and all `n` attempts fail
  and the final error is surfaced.

## Project Overview
SearcherKit is a pluggable search-agent runtime for retrieval-augmented tasks,
benchmark recipes, source plugins, Elasticsearch deployment, and multiple LLM
provider adapters.

Implementation priorities:
- Native concurrency and stable batch execution.
- Complete logging, including global logs and per-trace logs.
- Explicit error handling for recoverable, fatal, provider, source, and tool
  failures.
- Context/turn limit handling that asks the model to produce a final answer when
  limits are reached.
- Support both Hydra config and normal parameter passing. Avoid adding opaque
  `args` objects to internal APIs.
- Keep recipe, plugin, runtime, provider, and parser responsibilities separate.

## Current Layout
```text
src/searcherkit/
|-- __main__.py              # thin `python -m searcherkit` entry, delegates to CLI
|-- __init__.py
|-- agent/
|   |-- base.py              # agent protocol/base types
|   |-- search_agent.py      # SearchAgent and SearchAgentConfig
|   |-- react_agent.py       # ReAct-style example agent
|   `-- single_turn_agent.py # single-turn agent for judge/evaluate usage
|-- cli/
|   |-- main.py              # CLI dispatcher
|   |-- run.py               # run config/recipe
|   |-- evaluate.py          # evaluate saved outputs
|   |-- plugins.py           # plugin discovery/deploy entry
|   |-- inspect.py           # config validation
|   `-- config.py            # Hydra compose and ConfigStore registration
|-- common/
|   |-- config.py            # import/instantiate helpers
|   |-- dataloader.py        # generic dataloader
|   |-- errors.py            # project-level exception taxonomy
|   |-- json_schema.py       # JSON Schema helpers for tool interfaces
|   |-- log.py               # logging, trace logging, log context
|   |-- messages.py          # provider-agnostic message structures
|   |-- retry.py             # retry config and wrappers
|   `-- utils.py
|-- config/
|   |-- config.yaml          # packaged default run config
|   |-- searcherkit.yaml     # example config
|   |-- agent/               # agent config groups
|   |-- common/              # retry/dataloader config groups
|   |-- examples/
|   |-- llm/                 # provider/parser config groups
|   |-- plugins/
|   |-- runtime/
|   |-- sources/
|   |-- tools/
|   `-- training/
|-- llm/
|   |-- base.py              # Client/ClientConfig/get_client/provider configs
|   |-- openai.py            # OpenAI-compatible client
|   |-- dashscope.py         # DashScope adapter
|   |-- vllm.py              # vLLM adapter
|   |-- ollama.py            # Ollama adapter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ml-stat-Sustech/SearcherKit](https://github.com/ml-stat-Sustech/SearcherKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->

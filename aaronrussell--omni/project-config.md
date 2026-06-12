---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Omni is an Elixir library for interacting with LLM APIs across multiple providers. It separates three concerns:

- **Models** — data structs describing a specific LLM (loaded from JSON in `priv/models/`)
- **Providers** — authenticated HTTP layer (where to send requests, how to authenticate)
- **Dialects** — wire format translation (how to build request bodies, parse streaming events)

The relationship: ~4-5 dialects, ~20-30 providers (most speaking one dialect, though multi-model gateways like OpenCode Zen use multiple), hundreds of models (each belonging to one provider). The dialect is stored on each `%Model{}` struct, so dispatch is always per-model. All requests are streaming-first; `generate_text` is built on top of `stream_text`.

The agent system (`Omni.Agent`) lives in a separate package — [`omni_agent`](https://github.com/aaronrussell/omni_agent). This package is purely the stateless LLM API layer.

See the [Context Documents](#context-documents) section for when and how to use the detailed design docs.

## Build & Development Commands

```bash
mix compile                   # Compile the project
mix test                      # Run all tests
mix test --include live       # Run all tests including live API tests (needs API keys)
mix test path/to/test.exs     # Run a single test file
mix test path/to/test.exs:42  # Run a specific test (line number)
mix format                    # Format all code
mix format --check-formatted  # Check formatting without changing files
mix models.get                # Fetch model data from models.dev into priv/models/
```

`mix models.get` fetches model catalogs from [models.dev](https://models.dev) for each provider listed in its `@providers` attribute. It filters out deprecated models and those without tool use support. The JSON files in `priv/models/` are checked into the repo — run this task manually when model data needs refreshing.

## Dependencies

- **Req** (`~> 0.5`) — HTTP client (used for streaming requests to LLM APIs)
- **Peri** (`~> 0.6`) — Schema-based validation (used for option validation)
- **ExDoc** (dev only) — Documentation generation
- **Plug** (test only) — Required for `Req.Test` plug-based mocking

## Architecture

### Key design patterns

- **Streaming-first**: Every LLM request uses streaming HTTP via Req's `into: :self` async mode. The event pipeline is a lazy `Stream`: format parsing (SSE or NDJSON) → `Request.parse_event/2` (dialect `handle_event/1` + provider `modify_events/2`) → delta tuples. `StreamingResponse.new/2` builds the consumer pipeline via `Stream.transform/5`. The struct holds `stream` (the pipeline) and `cancel` (a zero-arity function). `Enumerable` yields `{event_type, data, partial_response}` tuples. Key functions: `on/3` (pipeline-composable side-effect handler), `text_stream/1`, `complete/1`, `cancel/1`. `:done` is only emitted when a stop reason was received; incomplete streams emit `{:error, :incomplete_stream}`.

- **Models are data, not modules**: `%Model{}` structs are loaded from `priv/models/*.json` at startup into `:persistent_term`. Models carry direct module references to their provider and dialect — the dialect is on the model, not derived from the provider, enabling multi-dialect providers.

- **Request building separated from execution**: `Request.build/3` validates options via Peri and returns a `%Req.Request{}` via dialect + provider composition. `Request.stream/3` executes and returns a `StreamingResponse`. The dialect transforms Omni types ↔ native JSON. The provider optionally modifies via `modify_body/3` and `modify_events/2`.

- **Two message roles only**: `:user` and `:assistant`. No `:tool` role — tool results are `Content.ToolResult` blocks inside user messages.

- **Recursive stream loop**: `Omni.Loop` handles tool auto-execution and structured output validation via recursive `Stream.flat_map`. `stream_text/3` always delegates to `Loop.stream/3`. Tools execute in parallel via `Tool.Runner.run/3`. `:max_steps` (default `:infinity`) caps rounds. When `:output` is set, the loop validates against the schema and retries up to 3 times on failure.

- **Single validation pass**: `Request.validate/2` merges universal + dialect option schemas, does a three-tier config merge (provider config ← app config ← call-site opts), and validates via Peri in one pass.

### Module layout

```
lib/omni.ex                         # Top-level API: generate_text, stream_text, get_model
lib/omni/
├── application.ex                  # Loads providers into :persistent_term at startup
├── model.ex                        # Model struct
├── context.ex                      # Context struct (system, messages, tools)
├── message.ex                      # Message struct (role, content blocks, timestamp)
├── response.ex                     # Response struct (wraps Message + metadata)
├── streaming_response.ex           # StreamingResponse + Enumerable impl
├── usage.ex                        # Usage struct (tokens + computed costs)
├── tool.ex                         # Tool struct, behaviour, use macro

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaronrussell/omni](https://github.com/aaronrussell/omni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->

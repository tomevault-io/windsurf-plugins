---
trigger: always_on
description: Guidance for LLM coding agents working on this repository.
---

# AGENTS.md

Guidance for LLM coding agents working on this repository.

## Project Overview

Conjure is an Elixir library for Anthropic Agent Skills with Claude models. It provides:

- **Unified Session API** - `Conjure.Session.chat/3` works across all backends
- **4 Execution Backends** - Local, Docker, Anthropic (hosted), Native (BEAM)
- **Skill Loading** - Parse SKILL.md files with YAML frontmatter, load `.skill` ZIP packages
- **Pluggable Storage** - Local, S3, Tigris backends for session file management
- **API-Agnostic** - No HTTP client bundled; consumers provide API callbacks

## Build & Development Commands

```bash
# Install dependencies
mix deps.get

# Run tests
mix test
mix test path/to/test.exs           # Specific file
mix test path/to/test.exs:42        # Specific line

# Code quality (all enforced in CI)
mix format                          # Auto-format
mix format --check-formatted        # Check formatting
mix credo --strict                  # Lint
mix dialyzer                        # Type checking
mix compile --warnings-as-errors    # Strict compilation

# Full CI check locally
mix format && mix credo --strict && mix compile --warnings-as-errors && mix test && mix dialyzer

# Generate documentation
mix docs
```

## Architecture

### Module Hierarchy

```
Conjure (Main API)
├── Session (Unified interface)
│   └── Backend (Behaviour)
│       ├── Local (System.cmd execution)
│       ├── Docker (Container execution)
│       ├── Anthropic (Skills API)
│       └── Native (BEAM execution)
├── Loader (SKILL.md parsing)
├── Registry (GenServer skill store)
├── Storage (Behaviour)
│   ├── Local (Temp directories)
│   ├── S3 (AWS)
│   └── Tigris (Fly.io)
└── NativeSkill (Behaviour for Elixir skills)
```

### Key Modules

| Module | Purpose |
|--------|---------|
| `Conjure` | Main public API entry point |
| `Conjure.Session` | Unified session interface, delegates to backends |
| `Conjure.Backend` | Behaviour defining unified interface for all backends |
| `Conjure.NativeSkill` | Behaviour for implementing skills as Elixir modules |
| `Conjure.Storage` | Behaviour for session storage backends |
| `Conjure.Loader` | SKILL.md parsing, ZIP loading, progressive disclosure |
| `Conjure.Registry` | GenServer-based skill registry with hot-reload |
| `Conjure.Conversation` | Tool-use conversation loop (Local/Docker) |
| `Conjure.Prompt` | Generates XML-formatted system prompt fragments |
| `Conjure.Tools` | Claude-compatible tool schemas (view, bash_tool, create_file, str_replace) |

### Directory Structure

```
lib/conjure/
├── backend/        # Backend implementations (local, docker, anthropic, native)
├── executor/       # Low-level tool execution (bash, view, create_file, str_replace)
├── conversation/   # Conversation loops including Anthropic pause-turn handling
├── storage/        # Storage backends (local, s3, tigris)
├── api/            # API helpers including Anthropic-specific beta headers
└── files/          # File operations for Anthropic Skills API
```

## Design Patterns

### Behaviour-Based Architecture

All pluggable components use Elixir behaviours:

```elixir
# Backend behaviour - all execution modes implement this
@callback backend_type() :: atom()
@callback new_session(skills :: term(), opts :: keyword()) :: Session.t()
@callback chat(Session.t(), String.t(), api_callback(), keyword()) :: chat_result()

# Storage behaviour - all storage backends implement this
@callback init(session_id, opts :: keyword()) :: {:ok, state} | {:error, term()}
@callback write(state, path, content) :: {:ok, file_ref} | {:error, term()}
@callback local_path(state) :: {:ok, path} | {:error, :not_supported}

# NativeSkill behaviour - for Elixir-based skills
@callback __skill_info__() :: skill_info()
@callback execute(String.t(), context()) :: result()  # Maps to bash_tool
@callback read(String.t(), context(), keyword()) :: result()  # Maps to view
```

### API-Agnostic Design

Conjure never makes HTTP calls. Consumers provide a callback:

```elixir
# The callback receives messages and returns {:ok, response} | {:error, term()}
{:ok, response, session} = Conjure.Session.chat(session, message, fn messages ->
  # Consumer's HTTP client call here
  Req.post("https://api.anthropic.com/v1/messages", json: build_body(messages))
end)
```

### Progressive Disclosure

Skills load with metadata only by default to save tokens:

```elixir
{:ok, skills} = Conjure.load("/path")  # Metadata only
{:ok, skill_with_body} = Conjure.load_body(skill)  # Load body when needed
{:ok, content} = Conjure.read_resource(skill, "scripts/helper.py")  # Load resources on demand
```

### OTP Compliance

- `Conjure.Registry` is a GenServer with supervision
- All backends handle errors gracefully with `{:ok, _} | {:error, _}` tuples
- Storage cleanup is reliable even on crashes

## Code Style

### General Conventions

- Use descriptive function names over comments
- Prefer pipeline operator (`|>`) for data transformations
- Pattern match in function heads over conditionals when possible
- Use `with` for chains of operations that can fail
- Return tagged tuples: `{:ok, result}` or `{:error, reason}`

### Module Structure

```elixir
defmodule Conjure.Example do
  @moduledoc """
  One-line description.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [holsee/conjure](https://github.com/holsee/conjure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

---
trigger: always_on
description: Graph-based agent orchestration for Elixir. Builds stateful, multi-step LLM workflows using nodes, edges, conditional routing, state reducers, human-in-the-loop interrupts, and checkpointing. Inspired by LangGraph, built on BEAM primitives.
---

# LangEx

Graph-based agent orchestration for Elixir. Builds stateful, multi-step LLM workflows using nodes, edges, conditional routing, state reducers, human-in-the-loop interrupts, and checkpointing. Inspired by LangGraph, built on BEAM primitives.

- **Version**: 0.5.0, **Elixir**: ~> 1.16
- **Deps**: `req`, `jason`, `telemetry`; optional `redix`, `postgrex`, `ecto_sql`
- **Test**: ExUnit with `mimic` for mocking

## Commands

```bash
mix deps.get                          # Install dependencies
mix compile --warnings-as-errors      # Compile (0 warnings required)
mix test                              # Run all tests
mix test path/to/test.exs:42          # Run specific test
mix format                            # Auto-format
mix format --check-formatted          # Check formatting
```

Always run `mix compile --warnings-as-errors` before considering work done. Zero warnings required.

## Architecture

```
LangEx (facade: invoke/3, stream/3)
├── Graph                             # Builder: new, add_node, add_edge, compile
│   ├── Graph.Compiled                # Compiled executable graph
│   ├── Graph.Pregel                  # Super-step execution engine (internal)
│   ├── Graph.State                   # State management with reducers
│   └── Graph.Stream                  # Lazy event streaming
├── LLM                               # Behaviour for provider adapters
│   ├── LLM.Anthropic                 # Claude adapter (streaming SSE)
│   │   ├── Anthropic.SSE             # SSE state machine (internal)
│   │   └── Anthropic.Formatter       # Message wire format (internal)
│   ├── LLM.OpenAI                    # GPT adapter
│   ├── LLM.Gemini                    # Gemini adapter
│   ├── LLM.Resilient                 # Retry wrapper with backoff
│   ├── LLM.ChatModel                 # Graph node helper for LLM calls
│   └── LLM.Registry                  # Provider resolution by model string
├── Tool                              # Tool/function definition struct
│   ├── Tool.Node                     # Graph node for parallel tool execution
│   └── Tool.Annotation               # Error recovery guidance for LLM
├── Message                           # Chat message types (Human, AI, System, Tool)
├── Checkpoint / Checkpointer         # Pause/resume with Redis or Postgres
├── Command / Send / Interrupt        # Graph control flow primitives
├── Config                            # Layered config resolution
├── ContextCompaction                 # Context window budget enforcement
└── Telemetry                         # Telemetry event definitions
```

### Behaviours

| Behaviour | Callbacks | Purpose |
|-----------|-----------|---------|
| `LangEx.LLM` | `chat/2`, `chat_with_usage/2` (optional) | LLM provider adapters |
| `LangEx.Checkpointer` | `save/2`, `load/1`, `list/2` | Checkpoint persistence backends |

### Key Design Decisions

- **No GenServers for domain state** -- graph state lives in function arguments and checkpoints, not processes
- **Pregel execution model** -- discrete super-steps with parallel node execution via `Task.Supervisor`
- **Process dictionary for interrupts** -- `Process.put(:lang_ex_resume, value)` enables the interrupt/resume mechanism
- **Reducers for state merging** -- each state key can have a custom reducer `(old, new) -> merged`

## Module Hierarchy

```
lib/lang_ex.ex                        → LangEx (facade)
lib/lang_ex/
├── command.ex                        → LangEx.Command
├── config.ex                        → LangEx.Config
├── context_compaction.ex            → LangEx.ContextCompaction
├── interrupt.ex                     → LangEx.Interrupt
├── send.ex                          → LangEx.Send
├── telemetry.ex                     → LangEx.Telemetry
├── checkpoint/
│   ├── checkpoint.ex                → LangEx.Checkpoint
│   ├── checkpointer.ex             → LangEx.Checkpointer (behaviour)
│   ├── postgres.ex                  → LangEx.Checkpointer.Postgres
│   └── redis.ex                     → LangEx.Checkpointer.Redis
├── graph/
│   ├── graph.ex                     → LangEx.Graph
│   ├── compiled_graph.ex            → LangEx.Graph.Compiled
│   ├── pregel.ex                    → LangEx.Graph.Pregel (@moduledoc false)
│   ├── state.ex                     → LangEx.Graph.State
│   └── stream.ex                    → LangEx.Graph.Stream
├── llm/
│   ├── llm.ex                       → LangEx.LLM (behaviour)
│   ├── anthropic.ex                 → LangEx.LLM.Anthropic
│   ├── anthropic/sse.ex             → LangEx.LLM.Anthropic.SSE (@moduledoc false)
│   ├── anthropic/formatter.ex       → LangEx.LLM.Anthropic.Formatter (@moduledoc false)
│   ├── openai.ex                    → LangEx.LLM.OpenAI
│   ├── gemini.ex                    → LangEx.LLM.Gemini
│   ├── resilient.ex                 → LangEx.LLM.Resilient
│   ├── chat_model.ex                → LangEx.LLM.ChatModel
│   └── chat_models.ex               → LangEx.LLM.Registry
├── message/
│   ├── message.ex                   → LangEx.Message (+ nested structs)
│   └── messages_state.ex            → LangEx.MessagesState
├── migration/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [surgeventures/lang_ex](https://github.com/surgeventures/lang_ex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->

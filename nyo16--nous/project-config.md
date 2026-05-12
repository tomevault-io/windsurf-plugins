---
trigger: always_on
description: Quick-reference for AI coding agents (Claude, Cursor, Copilot, Codex, etc.)
---

# AGENTS.md

Quick-reference for AI coding agents (Claude, Cursor, Copilot, Codex, etc.)
working with the **Nous** Elixir AI agent framework. This file is for agents
that want to *use* the library, not for agents maintaining the library
itself (see `CONTRIBUTING.md` and `docs/` for that). Conforms to
<https://agents.md>.

## What Nous is

Multi-provider LLM framework for Elixir/OTP. Provides:

- **One-shot LLM calls** (`Nous.generate_text/2,3`, `Nous.stream_text/2,3`)
- **Stateful agents** with tool-calling, memory, plugins (`Nous.new/2`, `Nous.run/2,3`)
- **Pluggable providers** — OpenAI, Anthropic, Gemini, Vertex AI, Groq, Mistral,
  OpenRouter, Together, Ollama, LM Studio, vLLM, SGLang, LlamaCpp, and a
  generic `custom:` adapter for any OpenAI-compatible endpoint
- **Tool system** — file ops, bash, web fetch + search, plus easy custom tools
- **Pluggable HTTP backend** (Req default, hackney alternative)
- **Streaming with backpressure** (hackney `:async, :once` pull mode)

## Minimal API surface (start here)

```elixir
# Drop-in: one-shot text generation
{:ok, text} = Nous.generate_text("openai:gpt-4o", "Explain GenServer in one sentence.")

# Streaming
{:ok, stream} = Nous.stream_text("anthropic:claude-sonnet-4-5", "Write a haiku")
Enum.each(stream, &IO.write/1)

# Stateful agent with tools
agent =
  Nous.new("openai:gpt-4o",
    tools: [Nous.Tools.FileRead, Nous.Tools.FileGrep],
    system_prompt: "You are a code reviewer."
  )

{:ok, result} = Nous.run(agent, "Find all TODOs in lib/")
# result.text, result.messages, result.usage

# Streaming agent run (text deltas only, no tool execution)
{:ok, stream} = Nous.run_stream(agent, "Summarize this repo")

# Streaming + tool execution in the same call (Nous 0.15.3+)
{:ok, result} = Nous.run(agent, "Search and summarize",
  stream: true,
  callbacks: %{
    on_llm_new_delta: fn _, t -> IO.write(t) end,
    on_llm_new_thinking_delta: fn _, t -> IO.write(["[thinking] ", t]) end
  }
)
```

That's 90% of what most apps need. Everything else is configuration.

## Provider quick-pick (model strings)

Format is `"<provider>:<model_id>"`. Pick one:

| If you want… | Use |
|---|---|
| Best general-purpose, high quality | `openai:gpt-4o` or `anthropic:claude-sonnet-4-5-20250929` |
| Cheap and fast | `groq:llama-3.1-70b-versatile` or `gemini:gemini-2.0-flash` |
| Local / no API key | `lmstudio:<your-loaded-model>` (default port 1234) |
| Local high-throughput inference | `vllm:<huggingface-id>` (default port 8000) |
| Local with structured generation | `sglang:<model>` (default port 30000) |
| Anything else with an OpenAI-compatible API | `custom:<model>` + `:base_url` opt |

Auth picks up the env var by convention: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`,
`GROQ_API_KEY`, `GEMINI_API_KEY`, `OPENROUTER_API_KEY`, etc. Local providers
don't need a key. Override per-call with `api_key:` opt.

## Key opts you'll actually use

```elixir
Nous.new("openai:gpt-4o",
  # LLM behavior
  system_prompt: "...",
  temperature: 0.7,
  max_tokens: 2_000,
  receive_timeout: 60_000,        # ms; 120_000 for local models

  # Tools (modules implementing Nous.Tool.Behaviour)
  tools: [Nous.Tools.Bash, MyApp.MyTool],

  # Memory backend (optional)
  memory: %{store: Nous.Memory.Store.ETS, opts: []},

  # Plugins (optional, composable)
  plugins: [Nous.Plugins.SubAgent, Nous.Plugins.HumanInTheLoop],

  # Resilience
  fallback: ["anthropic:claude-sonnet-4-5", "groq:llama-3.1-70b-versatile"],

  # Vendor-specific body params (vLLM/SGLang/LM Studio/llama.cpp)
  extra_body: %{top_k: 50, repetition_penalty: 1.1}
)
```

## Built-in tools

In `Nous.Tools.*`. The five most useful:

- **`Nous.Tools.Bash`** — execute shell commands (requires approval handler in production)
- **`Nous.Tools.FileRead`** / **`FileWrite`** / **`FileEdit`** — workspace-sandboxed file ops
- **`Nous.Tools.FileGlob`** / **`FileGrep`** — find files / search content
- **`Nous.Tools.WebFetch`** — fetch + extract text from a URL (SSRF-protected)
- **`Nous.Tools.TavilySearch`** / **`BraveSearch`** — web search

File tools enforce a workspace root. Default is `cwd`. Override per-agent:

```elixir
Nous.new("openai:gpt-4o",
  tools: [Nous.Tools.FileRead],
  deps: %{workspace_root: "/path/to/project"}
)
```

## Building a custom tool

```elixir
defmodule MyApp.WeatherTool do
  use Nous.Tool

  @impl Nous.Tool.Behaviour
  def name, do: "get_weather"

  @impl Nous.Tool.Behaviour
  def description, do: "Get current weather for a city"

  @impl Nous.Tool.Behaviour
  def parameters do
    %{
      "type" => "object",
      "properties" => %{
        "city" => %{"type" => "string", "description" => "City name"}
      },
      "required" => ["city"]
    }
  end

  @impl Nous.Tool.Behaviour
  def execute(%{"city" => city}, _ctx) do
    {:ok, "Weather in #{city}: 72°F, sunny"}
  end
end
```

Pass it in the `tools:` list. The `_ctx` arg gives access to `deps`,
the workspace root, and the approval handler. Use `Nous.Tool.Validator`
for input validation — it runs automatically when `validate_args: true`
(the default).

## HTTP backend (don't change unless you need to)

Default backend is `Nous.HTTP.Backend.Req` — Req on top of Finch. It's
faster under parallel batching than the alternative. Override only if:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyo16/nous](https://github.com/nyo16/nous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->

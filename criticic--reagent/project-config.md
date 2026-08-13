---
trigger: always_on
description: Do NOT delete or overwrite these files — they are intentionally maintained:
---

# CLAUDE.md — Development Guide for reagent

## Protected Files

Do NOT delete or overwrite these files — they are intentionally maintained:
- `README.md` — User-facing project documentation
- `CLAUDE.md` — This development guide
- `PITCH.md` — YC-style business pitch

## Project Structure

```
src/reagent/
  llm/          LLM abstraction (litellm-based, streaming, message types)
  agent/        Agent system (definitions, loop, orchestrator, registry)
  tool/         Tool system (base classes, registry, truncation)
    builtin/    General tools (shell, read, write, think, task, skill, dmail)
  re/           RE-specific tools (rizin, debugger, LIEF file info)
  model/        BinaryModel (observations, hypotheses, findings)
  context/      Context management (JSONL store, compaction, pruning, D-Mail)
  pty/          PTY process management (sessions, rolling buffers, process tree guard)
  skill/        Progressive skill loading system (SkillRegistry)
  session/      Session persistence and wire protocol
  tui/          Textual TUI (app, wire bridge)
  config/       Configuration (Pydantic models)
  cli.py        CLI entry point (analyze + tui commands)
agents/         Agent definitions (markdown with YAML frontmatter)
skills/         Skill files (rizin, gdb, frida command references)
tests/          Test suite
```

## Build & Test

```bash
uv sync                    # Install dependencies
uv run pytest              # Run tests
uv run reagent --help      # Run CLI
uv run reagent analyze BINARY -g "goal"         # Plain CLI mode
uv run reagent analyze BINARY -g "goal" -M      # Masked binary name (avoids agent bias)
uv run reagent tui BINARY -g "goal"             # Interactive TUI mode
uv run reagent tui BINARY -g "goal" -M          # TUI with masked binary name
```

## Environment Variables

Configure via `.env` file (loaded automatically) or shell environment:

| Variable | Description | Default |
|----------|-------------|---------|
| `ANTHROPIC_API_KEY` | Anthropic API key | — |
| `OPENAI_API_KEY` | OpenAI API key | — |
| `GEMINI_API_KEY` | Gemini API key | — |
| `REAGENT_MODEL` | Main model (litellm format) | `anthropic/claude-sonnet-4-5-20250929` |
| `REAGENT_FAST_MODEL` | Fast model for compaction | `anthropic/claude-haiku-4-5-20251001` |
| `REAGENT_CONTEXT_WINDOW` | Context window size | `200000` |
| `REAGENT_REASONING_EFFORT` | Reasoning effort (low/medium/high) | — |
| `REAGENT_FAST_REASONING_EFFORT` | Fast model reasoning effort | — |

Model names use litellm's `provider/model` prefix format. litellm reads API keys from env vars automatically — no need to pass them explicitly.

Examples:
```
# Anthropic
REAGENT_MODEL=anthropic/claude-sonnet-4-5-20250929

# Gemini
REAGENT_MODEL=gemini/gemini-3-flash-preview

# OpenAI
REAGENT_MODEL=openai/gpt-4o
```

## LLM Provider Architecture

- **`LiteLLMProvider`** — single unified provider class using `litellm.acompletion()`. litellm handles provider detection from the model string prefix and all SDK-specific format conversion internally.
- **`ChatProvider`** — runtime-checkable `Protocol` that `LiteLLMProvider` implements. Has `config` property and `stream()` async iterator method.
- **`create_provider(model, temperature, max_tokens, context_window)`** — factory function. No `api_key`, `base_url`, or `provider_type` args needed.
- Streaming uses litellm's `CustomStreamWrapper` which emits `ModelResponseStream` objects (OpenAI-format). `_chunk_to_dict()` normalizes these to our internal chunk dict format consumed by `streaming.py`.
- **Fast model** (`compact_provider`) — separate provider instance with `temperature=0.2` for context compaction. Threaded through `agent_loop()` -> `compact_fn()` -> `compact_context()`.

## Agent System

Agents are defined as markdown files in `agents/` with YAML frontmatter:

```yaml
---
name: agent_name
description: What this agent does
mode: primary | subagent
tools: [tool1, tool2, ...]
max_steps: 40
model: optional/model-override     # optional
temperature: 0.7                   # optional
---

System prompt content here...
```

Built-in agents:
- **orchestrator** (primary, 40 steps) — coordinates analysis, dispatches subagents, records findings
- **triage** (subagent, 15 steps) — quick recon: file format, arch, security features, strings
- **static** (subagent, 30 steps) — deep code analysis: decompilation, xrefs, control flow
- **dynamic** (subagent, 30 steps) — runtime verification: debugging, breakpoints, memory inspection
- **coding** (subagent, 15 steps) — computational verification: writes/runs Python scripts to decode, hash, keygen, etc.

The orchestrator dispatches subagents via `DispatchSubagentTool`. Each subagent gets its own temp Context, a tool registry subset, and the BinaryModel summary injected into its system prompt.

## Tool System

All tools inherit from `BaseTool` with Pydantic parameter models. Output goes through truncation (2000 lines / 50KB).

### Builtin Tools (7)

| Tool | Class | Description |
|------|-------|-------------|
| `shell` | `ShellTool` | Execute shell commands with process group isolation |
| `read_file` | `ReadFileTool` | Read file contents with offset/limit |
| `write_file` | `WriteFileTool` | Write content to files |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [criticic/reagent](https://github.com/criticic/reagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

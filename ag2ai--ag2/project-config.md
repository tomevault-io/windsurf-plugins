---
trigger: always_on
description: Before opening a PR, read and follow `.github/AI_POLICY.md`.
---

# AG2 Beta Development Guidelines

## AI-assisted contribution policy

Before opening a PR, read and follow `.github/AI_POLICY.md`.

- Do not open PRs with unverified AI-generated code or text.
- Ensure the PR description explains the real problem or use case and accurately reflects the diff.
- Include validation and testing information in the PR body.
- Be prepared to explain and revise the contribution in response to reviewer questions.

## Common rules

- Do not use `from __future__ import annotations`.
- Do not use global variables or top-level side-effect function calls unless the user explicitly allows it.
- For filesystem paths, use `pathlib.Path` internally. Public signatures should accept `str | os.PathLike[str]`.
- Top-level imports from `autogen.beta.*` are for common APIs that are broadly reusable across scenarios and core agent flows.
  Good: `autogen.beta.[Input]` — common structures usable in `await agent.ask(Input())` and as tool results.
  Bad: `autogen.beta.middleware.BaseMiddleware` — this is advanced/specialized and should be imported only when implementing custom middleware.
- Do not use function-level imports unless the user explicitly allows it.
  ```python
  # === BAD - import inside function ===
  def execute_tool():
      from .tool import Tool
      ...

  # === GOOD - top-level import ===
  from .tool import Tool

  def execute_tool():
      ...
  ```
- Do not create nested functions inside runtime execution paths.
  ```python
  # === BAD - function will be created each call ===
  def execute_tool():
      def _inner_function():
          pass

      _inner_function()

  # === GOOD - function created once, executed each call ===
  def execute_tool():
      _inner_function()

  def _inner_function():
      pass

  # === GOOD - decorator executed import time, so we can use closure functions here ===
  def decorator(func):
      def wrapper():
          return func()
      return wrapper
  ```
- Do not perform side effects in initialization methods. Apply side effects only at runtime.
  ```python
  # === BAD - create directory in initial method ===
  class KnowledgeStore:
      def __init__(self, path: str | os.PathLike[str]) -> None:
          self.path = Path(path)
          # side effect - directory creation
          self.path.parent.mkdir(parents=True, exist_ok=True)

      def run(self) -> None:
          ...

  # === GOOD - create directory in runtime method ===
  class KnowledgeStore:
      def __init__(self, path: str | os.PathLike[str]) -> None:
          self.path = Path(path)

      def run(self) -> None:
          self.path.parent.mkdir(parents=True, exist_ok=True)
          ...
  ```

## Package Structure

`autogen/beta/` is a protocol-driven async agent framework. Key modules:

| Module | Purpose | Key Exports |
|--------|---------|-------------|
| `agent.py` | Core agent loop and reply handling | `Agent`, `AgentReply` |
| `annotations.py` | Type annotations for dependency injection | `Context`, `Inject`, `Variable` |
| `context.py` | Runtime context (stream, dependencies, variables, prompt) | `Context` dataclass, `Stream` protocol |
| `stream.py` | In-memory event pub/sub | `MemoryStream`, `SubStream` |
| `events/` | Event types for the agent loop | `BaseEvent`, `ModelRequest`, `ModelResponse`, `ToolCallEvent`, `ToolResultEvent`, `Usage`, … |
| `config/` | LLM provider clients (see [below](#llm-provider-clients)) | `ModelConfig`, `LLMClient`, `AnthropicConfig`, `OpenAIConfig`, `GeminiConfig`, … |
| `tools/` | Tool system — builtin + user-defined | `tool`, `Toolkit`, `ToolResult`, `CodeExecutionTool`, `ShellTool`, `WebSearchTool`, … |
| `tools/subagents/` | Agent-to-agent delegation | `subagent_tool`, `run_task`, `persistent_stream`, `StreamFactory` |
| `eval/` | Offline evaluation framework | `run`, `scorer`, `EvalTarget`, `Suite`, `Task`, `Trace`, `RunResult`, `Feedback`, `BudgetThresholds`, plus prebuilts under `eval.scorers` |
| `middleware/` | Request/response interception | `BaseMiddleware`, `Middleware`, `LoggingMiddleware`, `RetryMiddleware`, `TokenLimiter`, `HistoryLimiter`, … |
| `response/` | Structured output validation | `ResponseSchema`, `PromptedSchema`, `ResponseProto`, `response_schema` |
| `history.py` | Conversation history storage | `History`, `Storage`, `MemoryStorage` |
| `hitl.py` | Human-in-the-loop hooks | — |
| `streams/` | Persistent stream backends (e.g. Redis) | — |

### Public API (`autogen.beta`)

Top-level modules:
- `autogen.beta` - top-level module with most basic functionality
- `autogen.beta.types` - Type aliases and constants
- `autogen.beta.config` - LLM provider clients (see [below](#llm-provider-clients))
- `autogen.beta.tools` - Tool system — builtin + user-defined (see [below](#builtin-tools))
- `autogen.beta.tools.subagents` - Agent-to-agent delegation (see [below](#subagent-delegation))
- `autogen.beta.testing` - Testing utilities
- `autogen.beta.middleware` - Request/response interception (see [below](#middleware))
- `autogen.beta.observer` - Reusable observer implementations
- `autogen.beta.eval` - Offline evaluation framework (datasets, scorers, runner, persistence)

Advanced modules:
- `autogen.beta.events` - Event types for the agent loop
- `autogen.beta.streams` - Persistent stream backends (e.g. Redis)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ag2ai/ag2](https://github.com/ag2ai/ag2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->

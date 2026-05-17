---
trigger: always_on
description: provides different execution modes: `run_async` for asynchronous execution
---

# AI Coding Assistant Context

This document provides context for AI coding assistants (Claude Code, Gemini
CLI, GitHub Copilot, Cursor, etc.) to understand the ADK Python project and
assist with development.

## Project Overview

The Agent Development Kit (ADK) is an open-source, code-first Python toolkit for
building, evaluating, and deploying sophisticated AI agents with flexibility and
control. While optimized for Gemini and the Google ecosystem, ADK is
model-agnostic, deployment-agnostic, and is built for compatibility with other
frameworks. ADK was designed to make agent development feel more like software
development, to make it easier for developers to create, deploy, and orchestrate
agentic architectures that range from simple tasks to complex workflows.

### Key Components

-   **Agent** - Blueprint defining identity, instructions, and tools
    (`LlmAgent`, `LoopAgent`, `ParallelAgent`, `SequentialAgent`, etc.)
-   **Runner** - Execution engine that orchestrates agent execution. It manages
    the 'Reason-Act' loop, processes messages within a session, generates
    events, calls LLMs, executes tools, and handles multi-agent coordination. It
    interacts with various services like session management, artifact storage,
    and memory, and integrates with application-wide plugins. The runner
    provides different execution modes: `run_async` for asynchronous execution
    in production, `run_live` for bidirectional streaming interaction, and
    `run` for synchronous execution suitable for local testing and debugging. At
    the end of each invocation, it can perform event compaction to manage
    session history size.
-   **Tool** - Functions/capabilities agents can call (Python functions, OpenAPI
    specs, MCP tools, Google API tools)
-   **Session** - Conversation state management (in-memory, Vertex AI,
    Spanner-backed)
-   **Memory** - Long-term recall across sessions

### How the Runner Works

The Runner is the stateless orchestration engine that manages agent execution.
It does not hold conversation history in memory; instead, it relies on services
like `SessionService`, `ArtifactService`, and `MemoryService` for persistence.

**Invocation Lifecycle:**

Each call to `runner.run_async()` or `runner.run()` processes a single user
turn, known as an **invocation**.

1.  **Session Retrieval:** When `run_async()` is called with a `session_id`, the
    runner fetches the session state, including all conversation events, from
    the `SessionService`.
2.  **Context Creation:** It creates an `InvocationContext` containing the
    session, the new user message, and references to persistence services.
3.  **Agent Execution:** The runner calls `agent.run_async()` with this context.
    The agent then enters its reason-act loop, which may involve:
    *   Calling an LLM for reasoning.
    *   Executing tools (function calling).
    *   Generating text or audio responses.
    *   Transferring control to sub-agents.
4.  **Event Streaming & Persistence:** Each step in the agent's execution (LLM
    call, tool call, tool response, model response) generates `Event` objects.
    The runner streams these events back to the caller and simultaneously
    appends them to the session via `SessionService`.
5.  **Invocation Completion:** Once the agent has produced its final response
    for the turn (e.g., a text response to the user), the agent's execution loop
    finishes.
6.  **Event Compaction:** If event compaction is configured, the runner may
    summarize older events in the session to manage context window limits,
    appending a `CompactedEvent` to the session.
7.  **Next Turn:** When the user sends another message, a new `run_async()`
    invocation begins, repeating the cycle by loading the session, which now
    includes the events from all prior turns.

## Project Architecture

Please refer to
[ADK Project Overview and Architecture](https://github.com/google/adk-python/blob/main/contributing/adk_project_overview_and_architecture.md)
for details.

### Source Structure

```
src/google/adk/
├── agents/          # Agent implementations (LlmAgent, LoopAgent, ParallelAgent, etc.)
├── runners.py       # Core Runner orchestration class
├── tools/           # Tool ecosystem (50+ files)
│   ├── google_api_tool/
│   ├── bigtable/, bigquery/, spanner/
│   ├── openapi_tool/
│   └── mcp_tool/    # Model Context Protocol
├── models/          # LLM integrations (Gemini, Anthropic, LiteLLM)
├── sessions/        # Session management (in-memory, Vertex AI, Spanner)
├── memory/          # Long-term memory services
├── evaluation/      # Evaluation framework (47 files)
├── cli/             # CLI tools and web UI
├── flows/           # Execution flow orchestration
├── a2a/             # Agent-to-Agent protocol
├── telemetry/       # Observability and tracing
└── utils/           # Utility functions
```

### Test Structure

```
tests/
├── unittests/       # 2600+ unit tests across 236+ files
│   ├── agents/
│   ├── tools/
│   ├── models/
│   ├── evaluation/
│   ├── a2a/
│   └── ...
└── integration/     # Integration tests
```

### ADK Live (Bidi-streaming)

-   ADK live feature can be accessed from runner.run_live(...) and corresponding
    FAST api endpoint.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [google/adk-python](https://github.com/google/adk-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->

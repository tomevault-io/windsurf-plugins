---
trigger: always_on
description: Agentspan is a distributed, durable runtime for AI agents. Agents survive crashes, scale across machines, and pause for human approval. Use Python SDK.
---

# Agentspan — Build Durable AI Agents

Agentspan is a distributed, durable runtime for AI agents. Agents survive crashes, scale across machines, and pause for human approval. Use Python SDK.

## Two Use Cases

**Developer building agents:** Define → deploy → serve → trigger by name. Long-lived, versioned, monitored.

**Autonomous agent building ephemeral agents:** Define → `rt.run(agent, prompt)` → get result → move on. No deploy. No serve. One call.

## Quickstart (Ephemeral — for autonomous agents)

```python
from agentspan.agents import Agent, AgentRuntime

agent = Agent(name="helper", model="openai/gpt-4o", instructions="You are a helpful assistant.")

with AgentRuntime() as rt:
    result = rt.run(agent, "What is quantum computing?")
    print(result.output["result"])   # String output
    # Or: result.print_result()      # Pretty-printed
```

`rt.run()` handles deploy + workers + execution internally. The agent is ephemeral — created for this task, discarded after.

## Production Pattern (for developers)

```python
from agentspan.agents import Agent, AgentRuntime

agent = Agent(name="helper", model="openai/gpt-4o", instructions="...")

if __name__ == "__main__":
    with AgentRuntime() as rt:
        # Deploy to server. CLI alternative (recommended for CI/CD):
        #   agentspan deploy my_module
        rt.deploy(agent)   # Push definition to server (idempotent)
        rt.serve(agent)    # Start workers, poll for tasks (blocks forever)
```

Trigger from outside: `agentspan run helper "What is quantum computing?"`

## Configuration

```python
# Default: reads AGENTSPAN_SERVER_URL from environment
rt = AgentRuntime()

# Explicit:
from agentspan.agents import AgentConfig
config = AgentConfig(server_url="http://localhost:6767/api", api_key="...")
rt = AgentRuntime(config=config)
```

Environment variables: `AGENTSPAN_SERVER_URL`, `AGENTSPAN_AUTH_KEY`, `AGENTSPAN_AUTH_SECRET`

## Agent

```python
Agent(
    name="my_agent",                    # Required. Unique. Alphanumeric + underscore/hyphen.
    model="openai/gpt-4o",             # "provider/model" format
    instructions="You are a ...",       # System prompt (str, callable, or PromptTemplate)
    tools=[my_tool],                    # List of @tool functions
    max_turns=25,                       # Max LLM iterations
    timeout_seconds=0,                  # 0 = no timeout
    max_tokens=None,                    # Max output tokens per LLM call
    temperature=None,                   # LLM temperature
    output_type=MyPydanticModel,        # Structured output (Pydantic model)
    planner=False,                      # Enable planning-first behavior
    thinking_budget_tokens=None,        # Extended reasoning token budget
    credentials=["API_KEY"],            # Credentials resolved from server
    metadata={"team": "backend"},       # Custom metadata
)
```

Model formats: `"openai/gpt-4o"`, `"anthropic/claude-sonnet-4-6"`, `"google_gemini/gemini-2.5-flash"`, `"claude-code/opus"`

### @agent Decorator

```python
from agentspan.agents import agent

@agent(model="openai/gpt-4o", tools=[search])
def researcher():
    """You are a research assistant. Find and summarize information."""

# Use like: rt.run(researcher, "Find info about quantum computing")
```

The docstring becomes the instructions.

## AgentResult

```python
result = rt.run(agent, "prompt")

result.output            # Dict: {"result": "..."} or agent-specific shape
result.output["result"]  # The text output (string)
result.status            # "COMPLETED", "FAILED", "TERMINATED", "TIMED_OUT"
result.execution_id      # Execution ID
result.error             # Error message if failed, else None
result.token_usage       # {"input_tokens": N, "output_tokens": N, ...}
result.finish_reason     # "stop", "length", "error", "cancelled", "timeout", "guardrail"
result.is_success        # True if COMPLETED
result.is_failed         # True if FAILED/TERMINATED
result.sub_results       # List of sub-agent results (multi-agent)
result.print_result()    # Pretty-print the output
```

## Error Handling

```python
result = rt.run(agent, "prompt")

if result.is_success:
    print(result.output["result"])
elif result.is_failed:
    print(f"Failed: {result.error}")
    print(f"Status: {result.status}")   # FAILED, TERMINATED, TIMED_OUT
    print(f"Reason: {result.finish_reason}")
```

For autonomous agents building ephemeral agents — always check `result.is_success` before using `result.output`.

## Tools

```python
from agentspan.agents import tool

@tool
def search(query: str) -> str:
    """Search the web for information."""
    return f"Results for: {query}"

@tool(approval_required=True, credentials=["API_KEY"])
def delete_file(path: str) -> str:
    """Delete a file. Requires human approval."""
    os.remove(path)
    return f"Deleted {path}"
```

Tool functions must have type hints and a docstring. The schema is generated automatically.

### ToolContext (dependency injection + shared state)

```python
from agentspan.agents import tool, ToolContext

@tool
def lookup(query: str, context: ToolContext) -> str:
    """Search with context."""
    wf_id = context.execution_id
    session = context.session_id

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentspan-ai/agentspan](https://github.com/agentspan-ai/agentspan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

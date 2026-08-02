---
trigger: always_on
description: Agents are the core building blocks of Agentex applications. Understanding what agents are and how they work is essential for building effective AI systems.
---

# Agent Concepts

Agents are the core building blocks of Agentex applications. Understanding what agents are and how they work is essential for building effective AI systems.

## What is an Agent?

In Agentex, think of **an Agent as just code** - specifically, it is just Python code that adheres to the Agent-to-Client Protocol (ACP). This allows clients to speak to any agent the same way regardless of the agent's complexity. This allows agent developers full unopinionated control over which libraries they use, which models they use, and how they implement their business logic.

An agent defines:

- **Handler functions** that respond to events (messages, task creation, etc.)
- **Business logic** for processing user requests
- **State management** for maintaining context across interactions
- **Integration code** for calling external services, APIs, or models

Think of an agent like a **web server endpoint** - it receives requests, processes them using your code, and returns responses. The difference is that agents are designed specifically for conversational AI and can maintain state across multiple interactions.

!!! note "Agents Are Not LLMs"
    An agent is **your application code**, not an LLM. While agents often call LLMs (like OpenAI's API), the agent itself is the Python code you write to orchestrate the conversation, manage state, and implement your business logic.

## Agent Relationships

!!! info "For Detailed Implementation"
    This section explains the architectural relationships between agents and other Agentex entities. For specific implementation patterns, refer to the [Agent Client Protocol guides](../agent_types/overview.md).

### Agent ↔ Tasks (Many-to-Many)

**A single agent can handle multiple tasks simultaneously, and a single task can involve multiple agents.**

#### Single Agent, Multiple Tasks

Your agent code runs independently for each task:

```python
@acp.on_message_send
async def handle_message_send(params: SendMessageParams):
    # This same function handles messages from many different tasks
    task_id = params.task.id  # Different for each conversation
    
    # Each task gets independent processing
    response = await process_for_task(task_id, params.content)
    return response
```

#### Multiple Agents, Single Task

Different agents can contribute to the same conversation:

```python
# Task "task_123" message history might include:
messages = [
    {"author": "USER", "content": "Analyze this data and create a report"},
    {"author": "AGENT", "content": "Starting analysis...", "agent_id": "data-analyst"},
    {"author": "AGENT", "content": "Analysis complete", "agent_id": "data-analyst"},
    {"author": "AGENT", "content": "Generating report...", "agent_id": "report-generator"},
    {"author": "AGENT", "content": "Report ready!", "agent_id": "report-generator"}
]
```

This enables **multi-agent workflows** where specialized agents collaborate on complex tasks.

### Agent ↔ State (One-to-One per Task)

**Each agent maintains its own isolated state for each task it's working on.**

#### Key Characteristics:

- **Scoped Storage**: State is isolated by `(task_id, agent_id)` pairs
- **Independent Operation**: Agents don't interfere with each other's state
- **Simple Management**: Each agent only needs to understand its own state
- **Parallel Safety**: Multiple agents can work simultaneously without conflicts

#### State Isolation Example:

```python
# Same task, different agents, separate states:

# Customer Support Agent state
support_state = {
    "customer_tier": "premium",
    "issue_category": "billing",
    "escalation_level": 1
}

# Technical Agent state  
tech_state = {
    "diagnostic_stage": "network_check",
    "test_results": ["ping_ok", "dns_ok"],
    "next_steps": ["check_firewall"]
}

# Both agents work on task_123 but maintain separate state
await adk.state.create(task_id="task_123", agent_id="support-agent", state=support_state)
await adk.state.create(task_id="task_123", agent_id="tech-agent", state=tech_state)
```

### Agent ↔ Messages (Many-to-Many)

**Agents can read all messages in a task and create messages marked with the AGENT author type.**

!!! note "Agent Identification in Messages"
    Currently, messages are not tagged with the specific agent that created them. In the future, Agentex will support providing the name of the agent that produced each message, enabling better tracking in multi-agent scenarios.

#### Message Creation:

```python
# Sync ACP - Return messages directly
@acp.on_message_send
async def handle_message_send(params: SendMessageParams):
    return TextContent(
        author=MessageAuthor.AGENT,
        content="Hello from the agent!"
    )

# Async ACP - Create messages explicitly
@acp.on_task_event_send
async def handle_event_send(params: SendEventParams):
    await adk.messages.create(
        task_id=params.task.id,
        content=TextContent(
            author=MessageAuthor.AGENT,
            content="Processing your request..."
        )
    )
```

#### Message Reading:

```python
# Agents can read all messages in a task
all_messages = await adk.messages.list(task_id=task_id)

# Filter messages by author type

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scaleapi/scale-agentex](https://github.com/scaleapi/scale-agentex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

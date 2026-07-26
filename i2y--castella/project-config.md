---
trigger: always_on
description: Castella provides comprehensive support for building AI agent interfaces with A2A (Agent-to-Agent) and A2UI (Agent-to-User Interface) protocols.
---

# Agent UI

Castella provides comprehensive support for building AI agent interfaces with A2A (Agent-to-Agent) and A2UI (Agent-to-User Interface) protocols.

!!! tip "MCP Support"
    Castella also supports [MCP (Model Context Protocol)](mcp.md) for AI agents to introspect and control UIs programmatically. See the [MCP documentation](mcp.md) for details.

## Quick Start

The simplest way to create a chat UI for an AI agent:

```python
from castella.agent import AgentChat

# Connect to an A2A-compatible agent
chat = AgentChat.from_a2a("http://localhost:8080")
chat.run()
```

Or with a custom handler:

```python
from castella.agent import AgentChat

chat = AgentChat(
    handler=lambda msg: f"You said: {msg}",
    title="My Bot",
    system_message="Welcome! How can I help you?",
)
chat.run()
```

## Installation

Agent features require the `agent` extra:

```bash
uv add "castella[agent,glfw]"
# or
pip install "castella[agent,glfw]"
```

## AgentChat

`AgentChat` is a high-level component that provides a complete chat interface with minimal setup.

### Constructor Options

```python
AgentChat(
    # Backend options (provide one)
    a2a_client: A2AClient | None = None,  # A2A client for remote agent
    handler: Callable[[str], str] | None = None,  # Custom message handler

    # UI options
    title: str | None = None,  # Window title
    placeholder: str = "Type a message...",  # Input placeholder
    system_message: str | None = None,  # Initial system message
    show_agent_card: bool = True,  # Show agent card for A2A agents
    a2ui_renderer: A2UIRenderer | None = None,  # Optional A2UI renderer

    # Window options (for run())
    width: int = 700,
    height: int = 550,
)
```

### Factory Methods

```python
# Create from A2A agent URL
chat = AgentChat.from_a2a("http://localhost:8080")
```

### Running the Chat

```python
chat.run()  # Opens a window and runs the chat
```

## A2A Client

The `A2AClient` class connects to A2A-compatible agents.

```python
from castella.a2a import A2AClient

# Connect to an agent
client = A2AClient("http://localhost:8080")

# Access agent metadata
print(f"Name: {client.name}")
print(f"Description: {client.description}")
print(f"Skills: {[s.name for s in client.skills]}")

# Send messages
response = client.ask("What's the weather in Tokyo?")
print(response)

# Async support
async def main():
    response = await client.ask_async("Hello!")
```

### Agent Card

```python
from castella.a2a import A2AClient

client = A2AClient("http://agent.example.com")

# Access the agent card
card = client.agent_card
print(f"Name: {card.name}")
print(f"Description: {card.description}")
print(f"Version: {card.version}")
print(f"URL: {card.url}")

# Skills
for skill in card.skills:
    print(f"  - {skill.name}: {skill.description}")
```

## A2UI Client

The `A2UIClient` class connects to A2A agents that support the A2UI extension, automatically handling protocol negotiation and UI generation.

```python
from castella import App
from castella.a2ui import A2UIClient, A2UIComponent, UserAction
from castella.frame import Frame

# Create client with action handler
def on_action(action: UserAction):
    print(f"Action: {action.name}")
    print(f"Source: {action.source_component_id}")
    print(f"Context: {action.context}")

client = A2UIClient("http://localhost:10002", on_action=on_action)

# Send message and get A2UI surface
surface = client.send("Find me restaurants in Tokyo")

if surface:
    # Render in Castella app
    App(Frame("Demo", 800, 600), A2UIComponent(surface)).run()
```

### Async Usage

```python
async def main():
    client = A2UIClient("http://localhost:10002")
    surface = await client.send_async("Hello!")

    # Send user actions back to agent
    if surface:
        action = UserAction(
            name="book_restaurant",
            source_component_id="book-button",
            context={"restaurant": "Sushi Place"}
        )
        await client.send_action_async(action)
```

### Features

- Connects to A2A Protocol agents with A2UI extension
- Handles A2UI extension negotiation automatically
- Extracts A2UI messages from A2A responses
- Integrates with `A2UIRenderer` for widget generation
- Supports both sync and async operations
- Maintains conversation context across messages

### Displaying Agent Card

```python
from castella.agent import AgentCardView, AgentListView
from castella.a2a import A2AClient

client = A2AClient("http://agent.example.com")

# Single agent card
card_view = AgentCardView(
    client.agent_card,
    show_skills=True,
    compact=False,
)

# List of agents
agents = [
    A2AClient("http://agent1.example.com").agent_card,
    A2AClient("http://agent2.example.com").agent_card,
]
list_view = AgentListView(agents)
```

## A2UI Renderer

The `A2UIRenderer` converts A2UI JSON specifications into native Castella widgets.

```python
from castella.a2ui import A2UIRenderer, UserAction

# Create renderer with action handler
def on_action(action: UserAction):
    print(f"Action: {action.name}")
    print(f"Source: {action.source_component_id}")
    print(f"Context: {action.context}")

renderer = A2UIRenderer(on_action=on_action)

# Render A2UI JSON

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [i2y/castella](https://github.com/i2y/castella) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

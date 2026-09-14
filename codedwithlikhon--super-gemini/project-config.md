---
trigger: always_on
description: Provides the result of a tool call execution.
---

# Copilot Instructions for Super-Gemini

## Project Overview
Super-Gemini is a local-first, agentic AI system that runs on Android through Termux, providing powerful terminal emulation and extensive Linux package management capabilities. It acts as a universal developer and productivity agent, automating tasks, scaffolding web apps, and controlling the environment using multiple runtimes (Bash, Python, Node.js).

# Agents
Source: https://docs.ag-ui.com/concepts/agents

Learn about agents in the Agent User Interaction Protocol

# Agents

Agents are the core components in the AG-UI protocol that process requests and
generate responses. They establish a standardized way for front-end applications
to communicate with AI services through a consistent interface, regardless of
the underlying implementation.

## What is an Agent?

In AG-UI, an agent is a class that:

1. Manages conversation state and message history
2. Processes incoming messages and context
3. Generates responses through an event-driven streaming interface
4. Follows a standardized protocol for communication

Agents can be implemented to connect with any AI service, including:

* Large language models (LLMs) like GPT-4 or Claude
* Custom AI systems
* Retrieval augmented generation (RAG) systems
* Multi-agent systems

## Agent Architecture

All agents in AG-UI extend the `AbstractAgent` class, which provides the
foundation for:

* State management
* Message history tracking
* Event stream processing
* Tool usage

```typescript
import { AbstractAgent } from "@ag-ui/client"

class MyAgent extends AbstractAgent {
  protected run(input: RunAgentInput): RunAgent {
    // Implementation details
  }
}
```

### Core Components

AG-UI agents have several key components:

1. **Configuration**: Agent ID, thread ID, and initial state
2. **Messages**: Conversation history with user and assistant messages
3. **State**: Structured data that persists across interactions
4. **Events**: Standardized messages for communication with clients
5. **Tools**: Functions that agents can use to interact with external systems

## Agent Types

AG-UI provides different agent implementations to suit various needs:

### AbstractAgent

The base class that all agents extend. It handles core event processing, state
management, and message history.

### HttpAgent

A concrete implementation that connects to remote AI services via HTTP:

```typescript
import { HttpAgent } from "@ag-ui/client"

const agent = new HttpAgent({
  url: "https://your-agent-endpoint.com/agent",
  headers: {
    Authorization: "Bearer your-api-key",
  },
})
```

### Custom Agents

You can create custom agents to integrate with any AI service by extending
`AbstractAgent`:

```typescript
class CustomAgent extends AbstractAgent {
  // Custom properties and methods

  protected run(input: RunAgentInput): RunAgent {
    // Implement the agent's logic
  }
}
```

## Implementing Agents

### Basic Implementation

To create a custom agent, extend the `AbstractAgent` class and implement the
required `run` method:

```typescript
import {
  AbstractAgent,
  RunAgent,
  RunAgentInput,
  EventType,
  BaseEvent,
} from "@ag-ui/client"
import { Observable } from "rxjs"

class SimpleAgent extends AbstractAgent {
  protected run(input: RunAgentInput): RunAgent {
    const { threadId, runId } = input

    return () =>
      new Observable<BaseEvent>((observer) => {
        // Emit RUN_STARTED event
        observer.next({
          type: EventType.RUN_STARTED,
          threadId,
          runId,
        })

        // Send a message
        const messageId = Date.now().toString()

        // Message start
        observer.next({
          type: EventType.TEXT_MESSAGE_START,
          messageId,
          role: "assistant",
        })

        // Message content
        observer.next({
          type: EventType.TEXT_MESSAGE_CONTENT,
          messageId,
          delta: "Hello, world!",
        })

        // Message end
        observer.next({
          type: EventType.TEXT_MESSAGE_END,
          messageId,
        })

        // Emit RUN_FINISHED event
        observer.next({
          type: EventType.RUN_FINISHED,
          threadId,
          runId,
        })

        // Complete the observable
        observer.complete()
      })
  }
}
```

## Agent Capabilities

Agents in the AG-UI protocol provide a rich set of capabilities that enable
sophisticated AI interactions:

### Interactive Communication

Agents establish bi-directional communication channels with front-end
applications through event streams. This enables:

* Real-time streaming responses character-by-character
* Immediate feedback loops between user and AI
* Progress indicators for long-running operations
* Structured data exchange in both directions

### Tool Usage

Agents can use tools to perform actions and access external resources.
Importantly, tools are defined and passed in from the front-end application to
the agent, allowing for a flexible and extensible system:

```typescript
// Tool definition
const confirmAction = {
  name: "confirmAction",
  description: "Ask the user to confirm a specific action before proceeding",
  parameters: {
    type: "object",
    properties: {
      action: {
        type: "string",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codedwithlikhon/Super-Gemini](https://github.com/codedwithlikhon/Super-Gemini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->

---
trigger: always_on
description: The `ToolLoopAgent` class in AI SDK v6 provides a framework for building agentic applications that can use tools iteratively to accomplish complex tasks.
---

# AI SDK v6 Agents Reference

## Overview

The `ToolLoopAgent` class in AI SDK v6 provides a framework for building agentic applications that can use tools iteratively to accomplish complex tasks.

## Installation

```bash
bun add ai @ai-sdk/anthropic zod
```

## Model Selection

Agents work with both direct providers and gateway:

```typescript
// Direct provider
import { anthropic } from "@ai-sdk/anthropic";
model: anthropic("claude-sonnet-4-5");

// Gateway (recommended for production)
import { gateway } from "ai";
model: gateway("anthropic/claude-sonnet-4-5");
```

## Basic Agent Setup

```typescript
import { ToolLoopAgent, tool, stepCountIs, gateway } from "ai";
import { z } from "zod";

const myAgent = new ToolLoopAgent({
  model: gateway("anthropic/claude-sonnet-4-5"),
  instructions: "You are a helpful assistant.",
  tools: {
    getData: tool({
      description: "Fetch data from API",
      inputSchema: z.object({
        query: z.string(),
      }),
      execute: async ({ query }) => {
        return { result: "data for " + query };
      },
    }),
  },
  stopWhen: stepCountIs(20),
});
```

## Configuration Options

| Parameter      | Type                              | Description                                                                      |
| -------------- | --------------------------------- | -------------------------------------------------------------------------------- |
| `model`        | `LanguageModel`                   | The AI model (e.g., `gateway('anthropic/claude-sonnet-4-5')` or direct provider) |
| `instructions` | `string`                          | System prompt defining agent behavior                                            |
| `tools`        | `Record<string, Tool>`            | Available tools the agent can call                                               |
| `stopWhen`     | `StopCondition`                   | When to terminate the agent loop                                                 |
| `toolChoice`   | `ToolChoice`                      | Controls tool usage: `'auto'`, `'required'`, `'none'`                            |
| `output`       | `Output<T>`                       | Optional structured output schema                                                |
| `prepareStep`  | `(step: StepInfo) => StepConfig`  | Dynamic per-step configuration                                                   |
| `prepareCall`  | `(call: CallInfo) => CallOptions` | Runtime options injection (e.g., for RAG)                                        |

## Stop Conditions

### Built-in Conditions

```typescript
import { stepCountIs, hasToolCall } from "ai";

// Stop after N steps
stopWhen: stepCountIs(20);

// Stop when specific tool is called
stopWhen: hasToolCall("finalAnswer");
```

### Custom Conditions

```typescript
const customStopCondition = ({ stepCount, totalTokens, cost, toolCalls }) => {
  // Stop if cost exceeds limit
  if (cost > 0.1) return true;

  // Stop if token limit exceeded
  if (totalTokens > 10000) return true;

  // Stop if specific condition met
  return toolCalls.some((tc) => tc.name === "complete");
};

const agent = new ToolLoopAgent({
  // ...
  stopWhen: customStopCondition,
});
```

## Execution Modes

### Non-Streaming

```typescript
const { text, toolCalls, usage } = await myAgent.generate({
  prompt: "Find and analyze user data",
});

console.log(text);
console.log(
  "Tools called:",
  toolCalls.map((tc) => tc.name)
);
console.log("Tokens used:", usage.totalTokens);
```

### Streaming

```typescript
const stream = myAgent.stream({ prompt: "Find and analyze user data" });

for await (const chunk of stream) {
  if (chunk.type === "text-delta") {
    process.stdout.write(chunk.text);
  } else if (chunk.type === "tool-call") {
    console.log("Calling tool:", chunk.name);
  } else if (chunk.type === "tool-result") {
    console.log("Tool result:", chunk.result);
  }
}
```

## API Route Integration

```typescript
// app/api/agent/route.ts
import { createAgentUIStreamResponse } from "ai";
import { myAgent } from "@/agents/my-agent";

export async function POST(request: Request) {
  const { messages } = await request.json();

  return createAgentUIStreamResponse({
    agent: myAgent,
    uiMessages: messages,
  });
}
```

## Dynamic Tool Control with prepareStep

Control which tools are available at each step:

```typescript
const phaseAgent = new ToolLoopAgent({
  model: anthropic("claude-sonnet-4-5"),
  instructions: "You are a research assistant.",
  tools: {
    search: searchTool,
    analyze: analyzeTool,
    summarize: summarizeTool,
  },
  prepareStep: ({ stepCount }) => {
    // Phase 1: Only search
    if (stepCount < 3) {
      return { toolChoice: { type: "tool", toolName: "search" } };
    }
    // Phase 2: Analyze results
    if (stepCount < 6) {
      return { toolChoice: { type: "tool", toolName: "analyze" } };
    }
    // Phase 3: Summarize
    return { toolChoice: { type: "tool", toolName: "summarize" } };
  },
  stopWhen: stepCountIs(10),
});
```

## RAG Integration with prepareCall

Inject context dynamically before each model call:

```typescript
const ragAgent = new ToolLoopAgent({
  model: anthropic("claude-sonnet-4-5"),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PanCodeInventory/PanelAgent](https://github.com/PanCodeInventory/PanelAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

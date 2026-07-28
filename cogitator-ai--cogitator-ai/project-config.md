---
trigger: always_on
description: > Patterns, configuration, and best practices for building agents
---

# Agents

> Patterns, configuration, and best practices for building agents

## Overview

An Agent in Cogitator is a configured LLM instance with:

- **Model** — The underlying LLM (Llama, GPT-4, Claude, Gemini, etc.)
- **Instructions** — System prompt defining behavior
- **Tools** — Capabilities the agent can use

Memory and sandbox are configured at the Cogitator runtime level, not on individual agents.

```typescript
interface AgentConfig {
  id?: string;
  name: string;
  description?: string;

  provider?: string; // Explicit provider override (e.g., 'openai' for OpenRouter)
  model: string; // 'ollama/llama3.3:70b', 'openai/gpt-4o'
  temperature?: number; // 0-2, default 0.7
  topP?: number; // 0-1
  maxTokens?: number; // Max output tokens
  stopSequences?: string[];

  instructions: string; // System prompt
  tools?: Tool[]; // Available tools
  responseFormat?: ResponseFormat; // Structured output

  maxIterations?: number; // Max tool use loops, default 10
  timeout?: number; // Max execution time in ms, default 120000
}
```

---

## Creating Agents

### Basic Agent

```typescript
import { Agent } from '@cogitator-ai/core';

const assistant = new Agent({
  name: 'assistant',
  model: 'ollama/llama3.3:latest',
  instructions: `You are a helpful assistant. Answer questions clearly and concisely.
                 If you don't know something, say so.`,
});
```

### Agent with Tools

```typescript
import { Agent, tool } from '@cogitator-ai/core';
import { z } from 'zod';

const searchWeb = tool({
  name: 'search_web',
  description: 'Search the internet for current information',
  parameters: z.object({
    query: z.string().describe('The search query'),
    limit: z.number().default(5).describe('Number of results'),
  }),
  execute: async ({ query, limit }) => {
    const results = await searchAPI.search(query, limit);
    return results.map((r) => ({ title: r.title, url: r.url, snippet: r.snippet }));
  },
});

const readUrl = tool({
  name: 'read_url',
  description: 'Read and extract content from a URL',
  parameters: z.object({
    url: z.string().url(),
  }),
  execute: async ({ url }) => {
    const content = await fetch(url).then((r) => r.text());
    return extractText(content);
  },
});

const researcher = new Agent({
  name: 'researcher',
  model: 'openai/gpt-4o',
  instructions: `You are a research assistant. Use your tools to find accurate,
                 up-to-date information. Always cite your sources.`,
  tools: [searchWeb, readUrl],
});
```

### Agent with Structured Output

```typescript
const analyzer = new Agent({
  name: 'analyzer',
  model: 'anthropic/claude-sonnet-4-5',
  instructions: 'Analyze the given text and extract structured information.',
  responseFormat: {
    type: 'json_schema',
    schema: z.object({
      summary: z.string(),
      sentiment: z.enum(['positive', 'negative', 'neutral']),
      keyPoints: z.array(z.string()),
      entities: z.array(
        z.object({
          name: z.string(),
          type: z.enum(['person', 'organization', 'location', 'other']),
        })
      ),
    }),
  },
});
```

### Agent with Persistent Memory

Memory is configured at the Cogitator runtime level, not on individual agents:

```typescript
import { Cogitator, Agent } from '@cogitator-ai/core';

const cog = new Cogitator({
  llm: {
    defaultModel: 'openai/gpt-4.1',
  },
  memory: {
    adapter: 'postgres',
    postgres: { connectionString: process.env.DATABASE_URL },
    embedding: {
      provider: 'openai',
      apiKey: process.env.OPENAI_API_KEY,
    },
    contextBuilder: {
      maxTokens: 8000,
      strategy: 'hybrid',
    },
  },
});

const personalAssistant = new Agent({
  name: 'personal-assistant',
  model: 'openai/gpt-4.1',
  instructions: `You are a personal assistant. Remember user preferences
                 and context from previous conversations.`,
});

await cog.run(personalAssistant, {
  input: 'Remember I prefer dark mode',
  threadId: 'user-alice',
});
```

---

## Agent Patterns

### 1. Planner Agent

Breaks down complex tasks into subtasks.

```typescript
const planner = new Agent({
  name: 'planner',
  model: 'openai/gpt-4o',
  temperature: 0.2,
  instructions: `You are a task planning agent. When given a complex task:
                 1. Analyze the requirements
                 2. Break it into specific, actionable subtasks
                 3. Identify dependencies between subtasks
                 4. Return a structured plan`,
  responseFormat: {
    type: 'json_schema',
    schema: z.object({
      goal: z.string(),
      subtasks: z.array(
        z.object({
          id: z.string(),
          description: z.string(),
          dependencies: z.array(z.string()),
          estimatedComplexity: z.enum(['low', 'medium', 'high']),
        })
      ),
    }),
  },
});
```

### 2. Executor Agent

Executes specific tasks with tools.

```typescript
const executor = new Agent({
  name: 'executor',
  model: 'anthropic/claude-sonnet-4-5',
  instructions: `You are a task execution agent. Execute the given task precisely.
                 Use tools when needed. Report success or failure clearly.`,
  tools: [fileRead, fileWrite, exec, webSearch],
  maxIterations: 20,
});
```

### 3. Critic Agent

Reviews and validates work.

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cogitator-ai/Cogitator-AI](https://github.com/cogitator-ai/Cogitator-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

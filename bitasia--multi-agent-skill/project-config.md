---
trigger: always_on
description: |
---


# Multi-Agent Workflow Engineering Skill

## Table of Contents
- [Quick Start](#quick-start)
- [Core Principle](#core-principle)
- [The Three Patterns](#the-three-patterns)
- [Design Principles](#design-principles)
- [Decision Framework](#decision-framework)
- [Failure Modes](#failure-modes)
- [Anti-Patterns](#anti-patterns)
- [Bundled Resources](#bundled-resources)

---

## Quick Start

Three patterns form the foundation of reliable multi-agent systems. Each uses TypeScript and Zod to enforce contracts at agent boundaries.

### 1. Typed Schema - Validate Agent Handoff Data

```typescript
import { z } from "zod";

const AgentHandoff = z.object({
  sourceAgent: z.string(),
  targetAgent: z.string(),
  timestamp: z.string().datetime(),
  payload: z.object({
    taskId: z.string().uuid(),
    status: z.enum(["pending", "in_progress", "completed", "failed"]),
    data: z.record(z.unknown()),
  }),
  metadata: z.object({
    correlationId: z.string().uuid(),
    retryCount: z.number().int().min(0).default(0),
    schemaVersion: z.literal("1.0"),
  }),
});

type AgentHandoff = z.infer<typeof AgentHandoff>;

// Validate on send
const validated = AgentHandoff.parse(outgoingData);

// Validate on receive
const received = AgentHandoff.parse(incomingData);
```

### 2. Action Schema - Constrain Agent Behavior

```typescript
import { z } from "zod";

const AgentAction = z.discriminatedUnion("type", [
  z.object({
    type: z.literal("delegate"),
    targetAgent: z.string(),
    task: z.string(),
    deadline: z.string().datetime().optional(),
  }),
  z.object({
    type: z.literal("respond"),
    content: z.string(),
    confidence: z.number().min(0).max(1),
  }),
  z.object({
    type: z.literal("escalate"),
    reason: z.string(),
    severity: z.enum(["low", "medium", "high", "critical"]),
  }),
  z.object({
    type: z.literal("retry"),
    originalAction: z.string(),
    backoffMs: z.number().int().positive(),
  }),
]);

type AgentAction = z.infer<typeof AgentAction>;

// Exhaustive matching ensures no action is missed
function handleAction(action: AgentAction): void {
  switch (action.type) {
    case "delegate":
      return routeToAgent(action.targetAgent, action.task);
    case "respond":
      return sendResponse(action.content);
    case "escalate":
      return notifyHuman(action.reason, action.severity);
    case "retry":
      return scheduleRetry(action.originalAction, action.backoffMs);
  }
}
```

### 3. MCP Contract - Define Tool Interfaces

```typescript
import { z } from "zod";

const SearchToolInput = z.object({
  query: z.string().min(1).max(500),
  filters: z.object({
    dateRange: z.object({
      from: z.string().datetime(),
      to: z.string().datetime(),
    }).optional(),
    maxResults: z.number().int().min(1).max(100).default(10),
  }).optional(),
});

const SearchToolOutput = z.object({
  results: z.array(z.object({
    id: z.string(),
    title: z.string(),
    relevance: z.number().min(0).max(1),
    snippet: z.string(),
  })),
  totalCount: z.number().int().min(0),
  executionMs: z.number().int().min(0),
});

const searchTool = {
  name: "search_documents",
  description: "Search documents with structured filters and validated results.",
  inputSchema: SearchToolInput,
  outputSchema: SearchToolOutput,
  handler: async (input: z.infer<typeof SearchToolInput>) => {
    const validated = SearchToolInput.parse(input);
    const results = await performSearch(validated);
    return SearchToolOutput.parse(results);
  },
};
```

---

## Core Principle

> **Treat agents like distributed systems, not chat flows.**

Most multi-agent failures stem from treating agents as cooperative conversation partners that naturally understand each other. In reality, agents are unreliable network services. This mental model shift changes everything about how you design multi-agent systems.

**Chat flow thinking** assumes agents share context, understand intent implicitly, pass data freely, and recover gracefully from errors. This works in demos but fails in production.

**Distributed systems thinking** assumes agents have no shared context, intent must be explicit in every message, data must be validated at every boundary, and failures are inevitable and must be handled. This is what makes multi-agent systems reliable.

The practical implications:

- **Explicit contracts**: Every agent boundary has a typed schema. No agent trusts data from another agent without validation.
- **Validation at boundaries**: Data is validated on send AND on receive. Schema mismatches surface immediately as errors, not as subtle downstream bugs.
- **Failure handling**: Every agent interaction has a timeout, a retry strategy, and an escalation path. No "happy path only" designs.
- **Observability**: Every message carries a correlation ID. Every state transition is logged. When something fails at 3 AM, you can trace exactly what happened.

---

## The Three Patterns

### Pattern 1: Typed Schemas

**Problem**: Agents pass unstructured data between each other. A planning agent emits a JSON blob, a coding agent consumes it, and somewhere in between a field name changes or a type shifts from string to number. The system doesn't crash -- it silently produces wrong results.

**Solution**: Define Zod schemas at every agent boundary. Validate data when sending and when receiving.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BITASIA/multi-agent-skill](https://github.com/BITASIA/multi-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

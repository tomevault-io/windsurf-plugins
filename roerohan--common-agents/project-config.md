---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Common Agents Framework** is a reusable multi-agent framework built on top of the Cloudflare Agents SDK. It provides structured, type-safe base classes for building multi-agent workflows on Cloudflare's infrastructure.

The framework offers two types of agents:
- **Permanent Agents**: Long-lived agents that maintain durable state (coordinators, routers, schedulers, knowledge bases, etc.)
- **Ephemeral Agents**: Self-destructing agents created for specific tasks (workers, processors, pipeline stages)

## Build Commands

```bash
# Build the project (compiles TypeScript to dist/)
npm run build

# Watch mode for development
npm run dev

# Type checking without emitting files
npm run typecheck

# Clean build artifacts
npm run clean
```

## Project Structure

```
src/
├── agents/base/           # Base agent classes (core framework)
│   ├── BaseAgent.ts       # Foundation for all agents with lifecycle hooks
│   ├── CoordinatorAgent.ts    # Result aggregation
│   ├── WorkerAgent.ts         # Single-task execution (ephemeral)
│   ├── FleetManagerAgent.ts   # Worker pool management
│   ├── RouterAgent.ts         # Request routing
│   ├── SchedulerAgent.ts      # Task scheduling with alarms
│   ├── KnowledgeBaseAgent.ts  # Data storage and retrieval
│   ├── TaskProcessorAgent.ts  # Complex task processing
│   ├── LLMToolAgent.ts        # LLM integration
│   ├── PipelineStageAgent.ts  # Pipeline stage execution
│   ├── QueueAgent.ts          # Task queue management
│   └── WatcherAgent.ts        # Resource monitoring
├── utils/
│   ├── types.ts           # Comprehensive type definitions
│   ├── id.ts              # ID generation utilities
│   ├── errors.ts          # Custom error classes
│   └── retry.ts           # Retry logic utilities
└── index.ts               # Main exports
```

## Architecture Principles

### Agent Permanence Model

**Permanent Agents**:
- Maintain durable state via `this.state` (automatically persisted)
- Live indefinitely unless explicitly destroyed
- Examples: CoordinatorAgent, KnowledgeBaseAgent, RouterAgent, SchedulerAgent, FleetManagerAgent, QueueAgent, WatcherAgent

**Ephemeral Agents**:
- Created for specific tasks, self-destruct after completion
- Call `this.state.reset()` in cleanup to mark for garbage collection
- Should not persist durable state
- Examples: WorkerAgent, TaskProcessorAgent, LLMToolAgent, PipelineStageAgent

### Lifecycle Model

All agents follow standardized lifecycle phases defined in `BaseAgent`:

1. **initialize()** - Setup state and resources (called in onStart)
2. **onBeforeTask()** - Pre-processing logic, validation
3. [Task execution]
4. **onAfterTask(success, error?)** - Post-processing, result handling
5. **cleanup()** - Resource deallocation (connections, locks, etc.)
6. **alarm()** - Scheduled/delayed actions (used by SchedulerAgent, WatcherAgent)

Ephemeral agents must call `cleanup()` before `state.reset()` to ensure proper resource release.

### State Management Rules

- **Permanent agents**: Use `this.setState()` to update durable state. State persists across invocations.
- **Ephemeral agents**: Use local instance variables for temporary data. Never rely on persisted state.
- All agents extend `BaseAgentState` which includes `created`, `lastUpdated`, `metadata`, and `agentName` fields.

### Communication Patterns

Agents communicate via:
- **Direct invocation**: `await getAgentByName(env.AGENT_NAMESPACE, agentId).method(args)`
- **Message passing**: Ephemeral workers send results to coordinator agents
- **State sharing**: Reading shared state from other agents

## Development Guidelines

### Creating Custom Agents

1. **Extend the appropriate base class** based on permanence and purpose:
   - For long-lived coordinators → `CoordinatorAgent`
   - For isolated task workers → `WorkerAgent`
   - For routing logic → `RouterAgent`
   - For scheduled tasks → `SchedulerAgent`
   - For data storage → `KnowledgeBaseAgent`
   - etc.

2. **Define specific state and task types**:
   ```typescript
   interface MyTaskData extends Task {
     data: { specific: string; fields: number; };
   }

   interface MyAgentState extends BaseAgentState {
     customField: string;
     results: ResultEntry[];
   }
   ```

3. **Override lifecycle hooks as needed**:
   - `initialize()` - Initial state setup
   - `onBeforeTask()` - Pre-task validation
   - `onAfterTask()` - Post-task cleanup
   - `cleanup()` - Resource deallocation (especially for ephemeral agents)

4. **For ephemeral agents**: Always call `await this.selfDestruct()` after task completion to trigger cleanup and state reset.

### Type Safety

- The framework is fully typed with TypeScript strict mode enabled
- All type definitions are in `src/utils/types.ts`
- When creating custom agents, define specific types for:
  - Agent state (extends `BaseAgentState`)
  - Task inputs (extends `Task`)
  - Task results (extends `TaskResult`)

### Error Handling

- Custom error classes are defined in `src/utils/errors.ts`
- Use the built-in retry utilities from `src/utils/retry.ts` for unreliable operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roerohan/common-agents](https://github.com/roerohan/common-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

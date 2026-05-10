---
trigger: always_on
description: This document provides guidance specifically for AI agents working on the Strands TypeScript SDK codebase. For human contributor guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md).
---

# Agent Development Guide - Strands TypeScript SDK

This document provides guidance specifically for AI agents working on the Strands TypeScript SDK codebase. For human contributor guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md).

## Purpose and Scope

**AGENTS.md** contains agent-specific repository information including:

- Directory structure with summaries of what is included in each directory
- Development workflow instructions for agents to follow when developing features
- Coding patterns and testing patterns to follow when writing code
- Style guidelines, organizational patterns, and best practices

**For human contributors**: See [CONTRIBUTING.md](CONTRIBUTING.md) for setup, testing, and contribution guidelines.

## Directory Structure

The repo is an npm workspace monorepo. The root `package.json` delegates all build/test/lint commands to the `strands-ts` workspace package.

```
sdk-typescript/
├── strands-ts/                   # SDK workspace package
│   ├── src/                      # All production code
│   │   ├── __fixtures__/         # Shared test fixtures (mocks, helpers)
│   │   ├── __tests__/            # Unit tests for root-level source files
│   │   │
│   │   ├── a2a/                  # Agent-to-agent protocol
│   │   │   ├── __tests__/
│   │   │   ├── a2a-agent.ts      # A2A agent client
│   │   │   ├── adapters.ts       # Strands/A2A type converters
│   │   │   ├── events.ts         # A2A streaming events
│   │   │   ├── executor.ts       # A2A executor
│   │   │   ├── express-server.ts # Express-based A2A server
│   │   │   ├── server.ts         # A2A server base
│   │   │   └── index.ts
│   │   │
│   │   ├── agent/                # Agent loop and streaming
│   │   │   ├── __tests__/
│   │   │   ├── agent.ts          # Core agent implementation
│   │   │   ├── agent-as-tool.ts  # Wrap agent as a tool
│   │   │   ├── printer.ts        # Agent output printing
│   │   │   └── snapshot.ts       # Agent state snapshots
│   │   │
│   │   ├── conversation-manager/ # Conversation history strategies
│   │   │   ├── __tests__/
│   │   │   ├── conversation-manager.ts
│   │   │   ├── null-conversation-manager.ts
│   │   │   ├── sliding-window-conversation-manager.ts
│   │   │   ├── summarizing-conversation-manager.ts
│   │   │   └── index.ts
│   │   │
│   │   ├── hooks/                # Hooks system for extensibility
│   │   │   ├── __tests__/
│   │   │   ├── events.ts
│   │   │   ├── registry.ts
│   │   │   ├── types.ts
│   │   │   └── index.ts
│   │   │
│   │   ├── logging/              # Structured logging
│   │   │   ├── __tests__/
│   │   │   ├── logger.ts
│   │   │   ├── warn-once.ts      # Dedupe warnings by message content
│   │   │   ├── types.ts
│   │   │   └── index.ts
│   │   │
│   │   ├── models/               # Model provider implementations
│   │   │   ├── __tests__/
│   │   │   ├── google/           # Google Gemini provider
│   │   │   ├── openai/           # OpenAI provider (Chat Completions + Responses API)
│   │   │   │   ├── __tests__/    # Unit tests (chat.test.ts, responses.test.ts)
│   │   │   │   ├── chat-adapter.ts
│   │   │   │   ├── responses-adapter.ts
│   │   │   │   ├── formatting.ts
│   │   │   │   ├── errors.ts
│   │   │   │   ├── model.ts
│   │   │   │   ├── types.ts
│   │   │   │   └── index.ts
│   │   │   ├── anthropic.ts      # Anthropic Claude
│   │   │   ├── bedrock.ts        # AWS Bedrock
│   │   │   ├── vercel.ts         # Vercel AI SDK
│   │   │   ├── defaults.ts       # Centralized model defaults + warning messages
│   │   │   ├── model.ts          # Base model interface
│   │   │   └── streaming.ts      # Streaming event types
│   │   │
│   │   ├── multiagent/           # Multi-agent orchestration
│   │   │   ├── __tests__/
│   │   │   ├── graph.ts          # Graph orchestrator (DAG)
│   │   │   ├── swarm.ts          # Swarm orchestrator (handoff)
│   │   │   ├── multiagent.ts     # Base multi-agent class
│   │   │   ├── nodes.ts          # Node types
│   │   │   ├── state.ts          # State management
│   │   │   ├── events.ts         # Streaming events
│   │   │   ├── edge.ts           # Edge definitions
│   │   │   ├── queue.ts          # Execution queue
│   │   │   ├── snapshot.ts       # Multi-agent snapshots
│   │   │   ├── plugins.ts        # Multi-agent plugins
│   │   │   └── index.ts
│   │   │
│   │   ├── plugins/              # Plugin system
│   │   │   ├── __tests__/
│   │   │   ├── plugin.ts
│   │   │   ├── registry.ts
│   │   │   ├── model-plugin.ts   # Clears agent messages after invocation when model is stateful
│   │   │   └── index.ts
│   │   │
│   │   ├── registry/             # Tool registry
│   │   │   ├── __tests__/
│   │   │   └── tool-registry.ts
│   │   │
│   │   ├── session/              # Session management
│   │   │   ├── __tests__/
│   │   │   ├── session-manager.ts
│   │   │   ├── storage.ts        # Storage interface
│   │   │   ├── file-storage.ts   # File-based storage
│   │   │   ├── s3-storage.ts     # S3 storage
│   │   │   ├── types.ts
│   │   │   ├── validation.ts
│   │   │   └── index.ts
│   │   │
│   │   ├── telemetry/            # OpenTelemetry tracing and metrics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-agents/sdk-typescript](https://github.com/strands-agents/sdk-typescript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

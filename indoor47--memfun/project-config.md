---
trigger: always_on
description: Memfun is an open-source autonomous coding/reasoning agent that combines three key technologies:
---

# Memfun - Autonomous Agent System

## Project Overview

Memfun is an open-source autonomous coding/reasoning agent that combines three key technologies:

1. **Memfun Runtime** - A pluggable, backend-agnostic infrastructure layer for deploying, coordinating, and scaling AI agents. Provides event-driven runtime, unified event bus, state management, MCP integration, and agent registry with tiered backends: In-Process (T0), SQLite (T1, default), Redis (T2), NATS JetStream (T3).

2. **DSPy RLM (Recursive Language Models)** - A DSPy module that lets LLMs explore large contexts through a sandboxed Python REPL with recursive sub-LLM calls. Handles contexts 2 orders of magnitude beyond native windows by separating variable space from token space.

3. **Agent Skills (agentskills.io)** - The open standard for portable AI agent capabilities. Memfun discovers, loads, activates, and executes skills; packages its own capabilities as portable skills; and synthesizes new skills from execution traces. Skills work across Claude Code, Codex CLI, Cursor, Gemini CLI, and 20+ other AI tools.

The vision: Build a production-ready autonomous agent that uses RLM patterns for massive codebase handling, a pluggable runtime for multi-agent orchestration, Agent Skills as the user-facing capability layer, and self-optimizes by discovering agent architectures and skills from RLM execution traces.

## Project Phase

This project is in the **production hardening** phase. Phases 1-4 are complete. Current focus: **Phase 5 (Production & Release)**.

Completed:
- Phase 1: Core foundation (memfun-core, protocols, T0+T1 backends, RuntimeBuilder, CI)
- Phase 2: Single-agent (RLM agent, LocalSandbox, MCP tools+gateway, CLI, 4 skills, trace collector)
- Phase 3: Multi-agent (T2 Redis + T3 NATS backends, AgentManager + Orchestrator, AGENT.md system, skill execution pipeline, Docker+Modal sandboxes, 4 more skills, 4 built-in agents, skills+agents as MCP tools, multi-agent CLI, conformance tests)
- Phase 4: Self-optimization (trace analysis pipeline, agent synthesis, MIPROv2 optimization, persistent memory, skill effectiveness tracking, marketplace foundation, interactive chat CLI)

In progress:
- Phase 5: Production & release (tests for Phase 4 modules, documentation, Docker, CI/CD, security audit, packaging)

## Monorepo Structure

```
memfun/
  packages/
    memfun-core/          # Shared types, config (memfun.toml), logging, errors
    memfun-tools/         # MCP tool server (FastMCP), repo map, code/git/fs/web tools
    memfun-agent/         # RLM agent, MCP Tool Bridge, trace collection
    memfun-runtime/       # Pluggable runtime: protocols (8), backends (T0-T3), BaseAgent, @agent
    memfun-optimizer/     # Self-optimization: trace analysis, agent synthesis, MIPROv2
    memfun-skills/        # Agent Skills runtime: discovery, loading, execution, synthesis
    memfun-cli/           # CLI app (Typer): setup wizard, commands, skill invocation, display
  skills/                 # Built-in Agent Skills (analyze-code, review-code, security-audit, etc.)
  agents/                 # Built-in agent definitions as AGENT.md (agent-architect, orchestrator, etc.)
  docker/                 # Docker Compose for NATS/Redis, agent Dockerfile
  docs/                   # MkDocs documentation site
  .github/workflows/      # CI/CD pipelines
```

## Agent Ecosystem

This project uses 18 specialized implementation agents organized into 9 groups:

### Core Infrastructure
| Agent | Purpose | Invoke With |
|-------|---------|-------------|
| `runtime-implementer` | Protocol interfaces, RuntimeContext, BaseAgent, @agent, T0 backend | `/runtime-implementer` |
| `sqlite-backend` | T1 SQLite backend: WAL-mode event bus, KV store, sessions, registry | `/sqlite-backend` |
| `event-bus-implementer` | T2 Redis and T3 NATS backends, migration system | `/event-bus-implementer` |

### Security
| Agent | Purpose | Invoke With |
|-------|---------|-------------|
| `security-auditor` | Security review, trust tiers, RBAC, audit trail, secret management | `/security-auditor` |
| `sandbox-implementer` | SandboxAdapter: Local/Docker/Modal sandbox backends, isolation | `/sandbox-implementer` |

### Agent System
| Agent | Purpose | Invoke With |
|-------|---------|-------------|
| `agent-system` | Agent definition format (AGENT.md), discovery, registration, delegation, orchestration | `/agent-system` |

### Intelligence Layer
| Agent | Purpose | Invoke With |
|-------|---------|-------------|
| `rlm-engine` | RLM coding agent, MCP Tool Bridge, trace collection, DSPy signatures | `/rlm-engine` |
| `trace-pipeline` | Self-optimization: trace analysis, agent synthesis, MIPROv2, A/B testing | `/trace-pipeline` |

### MCP & Tools
| Agent | Purpose | Invoke With |
|-------|---------|-------------|
| `mcp-gateway` | FastMCP 3.0 gateway, providers, transforms, agent-as-tool, skills-as-tool | `/mcp-gateway` |
| `code-tools` | MCP tools: filesystem, search (ripgrep/ast-grep), git, repo map | `/code-tools` |
| `web-tools` | MCP tools: web_fetch (URL->markdown), web_search (pluggable backends), SSRF prevention | `/web-tools` |

### Skills
| Agent | Purpose | Invoke With |
|-------|---------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [indoor47/memfun](https://github.com/indoor47/memfun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->

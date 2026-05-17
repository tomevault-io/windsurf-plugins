---
trigger: always_on
description: AutoBE is an AI-powered no-code system that automatically generates production-ready backend applications from natural language requirements. Users describe their backend needs through a chat interface, and AutoBE generates a complete TypeScript + NestJS + Prisma application.
---

# AutoBE Project Context for Claude Code

## What is AutoBE?

AutoBE is an AI-powered no-code system that automatically generates production-ready backend applications from natural language requirements. Users describe their backend needs through a chat interface, and AutoBE generates a complete TypeScript + NestJS + Prisma application.

**Core Capabilities**:
- 100% compilation guarantee
- Full type safety across the stack
- Comprehensive documentation (ERD, OpenAPI, API docs)
- Complete E2E test coverage
- Clean, maintainable implementation code

**Generated Outputs**:
Requirements Analysis → Database Schema (Prisma) → API Specification (OpenAPI) → E2E Tests → API Implementation → Type-Safe SDK

## Three Fundamental Concepts

### 1. Waterfall + Spiral Pipeline

AutoBE follows a 5-phase waterfall with internal spiral loops:

**Requirements** → **Analyze** → **Database** → **Interface** → **Test** → **Realize**

Each phase has 40+ specialized AI agents that collaborate. Failures trigger spiral loops that regenerate and correct until success.

### 2. Compiler-Driven Development

Three-tier validation ensures 100% compilation:

**AutoBE Prisma Compiler** → **AutoBE OpenAPI Compiler** → **TypeScript Compiler**

Compiler diagnostics feed back to AI agents, creating self-healing loops that automatically correct errors.

### 3. Vibe Coding

Conversation becomes software through automatic transformation:

**Conversation** → **Requirements** → **AST** → **Code** → **Application**

The entire pipeline is event-driven, with 65+ event types tracking progress in real-time.

## Documentation Index

Detailed documentation is organized by topic. **Always consult relevant documents before making changes.**

### Architecture & Design
- **[ARCHITECTURE.md](.ai/ARCHITECTURE.md)** - Overall system architecture, three paradigms, package structure
- **[STATE_MACHINE.md](.ai/STATE_MACHINE.md)** - Step counter pattern, automatic state invalidation
- **[AST_DESIGN.md](.ai/AST_DESIGN.md)** - Simplified AST philosophy for AI generation

### Agent System (`@autobe/agent`)
- **[AGENTICA_INTEGRATION.md](.ai/AGENTICA_INTEGRATION.md)** - MicroAgentica pattern, IPointer, function calling abstraction
- **[FUNCTION_CALLING.md](.ai/FUNCTION_CALLING.md)** - Facade pattern, LLM autonomous decision making
- **[AGENT_ORCHESTRATION.md](.ai/AGENT_ORCHESTRATION.md)** - Hierarchical orchestration, batch processing, self-healing loops
- **[AGENT_SYSTEM_PROMPTS.md](.ai/AGENT_SYSTEM_PROMPTS.md)** - System prompt design and editing guide ⭐ **Most Important!**
- **[AGENT_TOOLS.md](.ai/AGENT_TOOLS.md)** - Function calling tool definitions
- **[AGENT_HISTORIES.md](.ai/AGENT_HISTORIES.md)** - Context optimization with prompt caching

### Compiler System (`@autobe/compiler`)
- **[COMPILER_SYSTEM.md](.ai/COMPILER_SYSTEM.md)** - Three-tier validation, diagnostic generation, feedback loops

### Type System & Communication
- **[TYPE_SYSTEM.md](.ai/TYPE_SYSTEM.md)** - @autobe/interface, discriminated unions, Typia, TGrid RPC
- **[EVENT_SYSTEM.md](.ai/EVENT_SYSTEM.md)** - 65+ event types, type-safe mapper, WebSocket streaming
- **[RPC_SYSTEM.md](.ai/RPC_SYSTEM.md)** - WebSocket-based type-safe RPC communication

### Performance
- **[OPTIMIZATION.md](.ai/OPTIMIZATION.md)** - Prompt caching, batch processing, concurrency control

### Frontend & UI (`@autobe/ui`, Website, Apps)
- **[FRONTEND_SYSTEM.md](.ai/FRONTEND_SYSTEM.md)** - UI components, real-time communication, session management

### Development
- **[DEVELOPMENT_GUIDE.md](.ai/DEVELOPMENT_GUIDE.md)** - Adding features, debugging, code navigation
- **[BEST_PRACTICES.md](.ai/BEST_PRACTICES.md)** - System prompt editing, optimization, troubleshooting

## Quick Start for Claude Code

### First Time Working on AutoBE
1. Read [ARCHITECTURE.md](.ai/ARCHITECTURE.md) - Understand system fundamentals
2. Read [AGENTICA_INTEGRATION.md](.ai/AGENTICA_INTEGRATION.md) - Understand agent structure
3. Read package-specific documentation for your work area

### Editing System Prompts (Most Important!)

**Absolute Rule**: User instructions are absolute. Claude Code must NEVER modify, reduce, or omit user commands based on its own judgment.

**Before Editing**:
1. **Read [AGENT_SYSTEM_PROMPTS.md](.ai/AGENT_SYSTEM_PROMPTS.md)** completely
2. Read the target prompt file fully
3. Review related Orchestrator code (`packages/agent/src/orchestrate/`)
4. Review related Tool definitions
5. Review related History transformers

**When Editing**:
- Integrate naturally into existing storyline
- Write clear, specific instructions
- Include rich examples
- Specify constraints explicitly
- Use positive directives ("do X" not "don't do Y")

**After Editing - CRITICAL**:
1. **STOP** - Your editing task is complete
2. Do **NOT** run `pnpm run build:prompt` unless user explicitly requested it
3. Do **NOT** run `pnpm run build` or `pnpm run test`
4. Do **NOT** run `git commit` or any git commands
5. Do **NOT** suggest or execute ANY additional steps

**The User Will Decide**:
- Whether to build the prompts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wrtnlabs/autobe](https://github.com/wrtnlabs/autobe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->

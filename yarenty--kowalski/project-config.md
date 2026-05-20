---
trigger: always_on
description: > **READ THIS FIRST**: This file serves as the single source of truth for any AI agent (Claude, Gemini, Cursor, etc.) working on the Kowalski repository. It aggregates architectural context, development workflows, and behavioral guidelines.
---

# Kowalski AI Agent Documentation

> **READ THIS FIRST**: This file serves as the single source of truth for any AI agent (Claude, Gemini, Cursor, etc.) working on the Kowalski repository. It aggregates architectural context, development workflows, and behavioral guidelines.

## Table of Contents
1. [Philosophy & Core Principles](#1-philosophy--core-principles)
2. [Project Identity](#2-project-identity)
3. [Architecture & Design Principles](#3-architecture--design-principles)
4. [Technology Stack](#4-technology-stack)
5. [Repository Structure](#5-repository-structure)
6. [Development Workflows](#6-development-workflows)
7. [Quality Standards](#7-quality-standards)
8. [Critical Rules & Protocols](#8-critical-rules--protocols)
9. [Implementation Status](#9-implementation-status)
10. [Common AI Tasks](#10-common-ai-tasks)

---

## 1. Philosophy & Core Principles

### Core Philosophy
- **Incremental progress over big bangs**: Break complex tasks into manageable stages
- **Learn from existing code**: Understand patterns before implementing new features
- **Clear intent over clever code**: Prioritize readability and maintainability
- **Simple over complex**: Keep implementations straightforward - prioritize solving problems over architectural complexity

### The Eight Honors and Eight Shames
| **Shame** | **Honor** |
|-----------|-----------|
| Guessing APIs | Careful research and documentation reading |
| Vague execution | Seeking confirmation before major changes |
| Assuming business logic | Human verification of requirements |
| Creating new interfaces | Reusing existing, proven patterns |
| Skipping validation | Proactive testing and error handling |
| Breaking architecture | Following established specifications |
| Pretending to understand | Honest acknowledgment of uncertainty |
| Blind modification | Careful, incremental refactoring |

### SOLID Principles Integration
Our codebase follows SOLID principles to ensure maintainable, scalable software.

**Quick Reference**: See [`tools/solid_principles_quick_reference.md`](tools/solid_principles_quick_reference.md) for essential patterns and checklists.

**Detailed Guide**: See [`tools/solid_principles_guide.md`](tools/solid_principles_guide.md) for comprehensive examples and implementation strategies.

#### Core SOLID Guidelines for AI Development
- **Single Responsibility (SRP)**: Before adding functionality, ask "Does this belong here?"
- **Open/Closed (OCP)**: Extend behavior through new classes/modules, not modifications
- **Liskov Substitution (LSP)**: Ensure any subclass can replace its parent without breaking functionality
- **Interface Segregation (ISP)**: Design small, specific interfaces rather than large, monolithic ones
- **Dependency Inversion (DIP)**: Inject dependencies rather than creating them directly

---

## 2. Project Identity

**Name**: Kowalski  
**Release line**: **1.2.0** (workspace; see root `Cargo.toml` and `CHANGELOG.md`).  
**Purpose**: A Rust-native multi-agent framework: **`kowalski-core`** (agents, LLM, memory, MCP client), **`kowalski-cli`** (REPL + operators, extensions, **`agent-app`**), **`kowalski`** (HTTP **`/api/*`** server), optional **`kowalski-mcp-datafusion`**, Vue **`ui/`**, optional PostgreSQL (**pgvector**, **Apache AGE**).  
**Core Value Proposition**: Modular, extensible deployment with MCP-first tools and federation-oriented APIs.  
**Primary Mechanism**: `TemplateAgent` + pluggable tools (built-in + MCP), Ollama/OpenAI-compatible providers.  
**Target Users**: Developers building operator-run or embedded agent systems on CPU-friendly stacks.  

### Business Context
- **Problem Solved**: Complexity in building and managing secure, federated multi-agent LLM systems.
- **Success Metrics**: Extensibility of tools, performance of async operations, successful federated task execution.
- **Key Constraints**: Rust-based architecture, efficient execution, secure multi-party computation.

---

## 3. Architecture & Design Principles

### Architectural Patterns
- **Actor Model**: Agent abstractions and isolated execution contexts
- **Generic Architecture**: Multi-purpose agent abstraction built on extensible configurations
- **Pluggable Architecture**: Extensible toolchain via MCP (upcoming) support

### Design Patterns in Use
- **Repository Pattern**: For data access abstraction
- **Factory Pattern**: For object creation
- **Strategy Pattern**: For algorithm selection
- **Observer Pattern**: For event handling
- **Message Passing Pattern**: For agent-to-agent communication
- **Plugin Pattern**: For dynamic tool integration

### Tool sources (where capabilities live)

All “do something with the outside world” behavior should align with **one model**, documented in [`kowalski-core/AGENTS.md`](kowalski-core/AGENTS.md) (**Tool execution model**):

1. **In-repo MCP servers** you ship and register (e.g. [`kowalski-mcp-datafusion`](kowalski-mcp-datafusion/)).
2. **External MCP** reached via gateways or catalogs (e.g. [Docker MCP Toolkit](https://docs.docker.com/ai/mcp-catalog-and-toolkit/toolkit/) — profiles, OAuth, catalog).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yarenty/kowalski](https://github.com/yarenty/kowalski) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

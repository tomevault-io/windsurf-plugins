---
trigger: always_on
description: This document provides essential context for AI assistants working on the [In the Loop](https://github.com/runtimed/intheloop) project.
---

# AI Agent Development Context

This document provides essential context for AI assistants working on the [In the Loop](https://github.com/runtimed/intheloop) project.

Current work state and development priorities. What works, what's experimental, what needs improvement. Last updated: October 2025.

**Development Workflow**: The user typically runs the integrated development server (`pnpm dev`) which handles both frontend and backend. For checking work, run type checks first, followed by lints, tests, and UI builds. The iframe outputs server runs separately (`pnpm dev:iframe`).

## Project Overview

In the Loop is a real-time collaborative notebook system built on event-sourced architecture using LiveStore. It supports multiple runtime paradigms: external runtime agents via `@runt` JSR packages, and in-browser runtime agents for HTML and Python (Pyodide) that execute directly in the web client.

**Current Status**: A working system deployed at https://app.runt.run. Features real-time collaboration, persistent execution outputs, and integrated AI capabilities. The system is stable for experimentation and real usage while actively developed.

## Architecture

### Monorepo Structure

In the Loop is organized as a monorepo with four core packages and a unified web application:

- **Packages** (`packages/`): Core libraries published to npm/JSR for runtime agent development
- **Web Client** (`src/`): React-based notebook interface with integrated backend
- **Backend** (`backend/`): Cloudflare Worker serving API and sync functionality
- **Runtime Agents**: External (via @runt JSR packages) and in-browser (HTML/Python) execution

### Core Packages

- **`@runtimed/schema`**: LiveStore schema definitions with full type inference across packages
- **`@runtimed/agent-core`**: Runtime agent framework with artifact storage and observability
- **`@runtimed/ai-core`**: Multi-provider AI integration (OpenAI, Ollama, Groq) with tool calling
- **`@runtimed/pyodide-runtime`**: In-browser Python runtime with scientific computing stack

### Key Dependencies

- **LiveStore**: Event-sourcing library for local-first collaborative applications
- **Effect**: Functional programming library for TypeScript with precise error handling
- **React**: UI framework with CodeMirror editors and Radix UI components
- **Cloudflare Workers**: Runtime for production deployment with D1/R2 storage

## Current Working State

### What's Working ✅

- ✅ **Event-sourced architecture** - All changes preserved through LiveStore events
- ✅ **Three runtime paradigms** - External agents, in-browser HTML, in-browser Python
- ✅ **Real-time collaboration** - Multiple users editing simultaneously without conflicts
- ✅ **Rich output system** - matplotlib plots, pandas DataFrames, terminal colors, images
- ✅ **AI integration** - Context-aware AI sees both code and execution results
- ✅ **Tool calling system** - AI can create, modify, and execute cells with approval
- ✅ **Persistent computation** - Outputs survive browser crashes and network drops
- ✅ **Mobile responsiveness** - Notebook editing works on mobile devices
- ✅ **Offline-first operation** - Local-first with sync when connected
- ✅ **Production deployment** - Unified Cloudflare Worker at https://app.runt.run
- ✅ **Authentication system** - OIDC OAuth with fallback token authentication
- ✅ **Package caching** - Pyodide pre-loads scientific stack for faster startup
- ✅ **Artifact service** - Large outputs stored in R2 with proper media types
- ✅ **Integrated development** - Single `pnpm dev` command runs frontend+backend

### Core Architecture Constraints

- `NOTEBOOK_ID = STORE_ID`: Each notebook has its own LiveStore database instance
- **Event-sourced state**: All mutations flow through LiveStore events, never direct state changes
- **Reactive execution**: `executionRequested` → `executionAssigned` → `executionStarted` → `executionCompleted`
- **Package workspace linking**: All packages use `workspace:*` for local development
- **Session-based runtimes**: Each restart gets unique `sessionId` for conflict resolution
- **Soft shutdown model**: Runtime agents can stop without affecting notebook state
- **Artifact-first outputs**: Large data stored as artifacts with proper Content-Type headers

### Runtime-Notebook Relationship

**Multiple Runtime Support**: Notebooks can have external agents, in-browser agents, or both, but typically one active runtime at a time for execution coherence.

**Runtime Lifecycle**:

- Notebook created → No runtime (user chooses which to start)
- External agent → Connects via WebSocket with API key authentication
- In-browser agent → Launches directly in browser tab sharing LiveStore
- Runtime restart → Brief overlap during handoff, then cleanup

**Current Implementation**: Manual runtime selection with UI-driven startup. Automatic orchestration and health monitoring planned but not implemented.

## Development Commands

```bash
# Setup
pnpm install         # Install dependencies
cp .env.example .env # Copy environment files
cp .dev.vars.example .dev.vars

# Development (integrated server)
pnpm dev            # Frontend + backend at http://localhost:5173

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runtimed/intheloop](https://github.com/runtimed/intheloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

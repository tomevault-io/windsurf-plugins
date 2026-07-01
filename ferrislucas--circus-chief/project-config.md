---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Circus Chief is a local-first web application for managing AI coding agent sessions with a visual canvas for artifacts. It supports **Claude Code** agents (via Anthropic API, AWS Bedrock, Google Vertex AI, z.ai, or any Anthropic-compatible endpoint), **OpenAI Codex** agents, and **Google Gemini CLI** agents. Users can view/manage sessions, share a visual canvas (images, documents, data), and interact with active coding sessions from a web browser.

## Common Commands

```bash
# Development - start both frontend and backend
yarn dev

# Run all unit tests
yarn test

# Run tests for a specific package
yarn workspace @circuschief/server test
yarn workspace @circuschief/web test

# Run a single test file (vitest)
yarn workspace @circuschief/server test src/db/SessionRepository.test.js
yarn workspace @circuschief/web test src/stores/ui.test.js

# E2E tests with Playwright
./scripts/pw.sh test                           # Run all E2E tests
./scripts/pw.sh test --grep="login"            # Filter by test name
./scripts/pw.sh debug tests/e2e/auth.spec.ts   # Debug mode (headed)

# Linting
yarn lint

# Build for production
yarn build
```

## Architecture

### Monorepo Structure (Yarn Workspaces)

- **packages/server** - Express backend with WebSocket support
- **packages/web** - Vue.js 3 frontend (Vite, Pinia, Vue Router)
- **packages/shared** - Shared types, constants, and Zod contracts

### Server Package (`@circuschief/server`)

- `src/index.js` - Entry point, starts HTTP server
- `src/app.js` - Express app configuration
- `src/websocket.js` - WebSocket server setup
- `src/api/` - REST API routes (projects, sessions, canvas, git, providers)
- `src/agents/` - Agent adapter pattern for multi-backend support
  - `AgentGateway.js` - Factory/registry for agent adapters
  - `BaseAgent.js` - Abstract base class for agent adapters
  - `adapters/ClaudeCodeAdapter.js` - Claude Code agent (Anthropic)
  - `adapters/CodexAdapter.js` - OpenAI Codex agent (OpenAI)
  - `adapters/GeminiAdapter.js` - Google Gemini CLI agent (Google)
  - `adapters/cliUtils.js` - Shared CLI utilities (prompt composition) used by Codex and Gemini adapters
- `src/db/` - SQLite repositories using better-sqlite3
  - `BaseRepository.js` - Abstract base with CRUD operations
  - Repository pattern: `ProjectRepository`, `SessionRepository`, `MessageRepository`, `CanvasItemRepository`, `SessionNoteRepository`
- `src/services/` - Business logic (sessionManager, canvasStore, gitService, diffService)
- `src/ws/` - WebSocket manager for real-time updates

### Web Package (`@circuschief/web`)

- `src/router.js` - Vue Router configuration
- `src/views/` - Page components (ProjectListView, SessionListView, SessionDetailView, etc.)
- `src/components/` - Reusable components (CanvasTab, ConversationTab, NotesTab, ChangesTab, ToastContainer)
- `src/stores/` - Pinia stores (projects, sessions, canvas, ui)

### Shared Package (`@circuschief/shared`)

- `src/types.js` - JSDoc type definitions and model lists (`CLAUDE_MODELS`, `OPENAI_MODELS`, `GEMINI_MODELS`)
- `src/protocol.js` - WebSocket message type definitions
- `src/constants.js` - Shared constants and enums
- `src/contracts/` - Zod validation schemas for API contracts (including `providers.js` for provider kinds: `anthropic` | `openai` | `google`)

### Communication Pattern

- **HTTP REST API** - CRUD operations, initial state fetching
- **WebSocket** - Real-time streaming (conversation updates, session status changes, new canvas items)

### Database

SQLite database with these main tables:
- `projects` - Project definitions with working directories
- `sessions` - Agent sessions (status: starting/running/waiting/stopped/error)
- `providers` - Configured AI providers (kind: `anthropic`, `openai`, or `google`)
- `provider_models` - Models registered per provider
- `conversation_messages` - Chat history per session
- `canvas_items` - Images, markdown, text, JSON artifacts
- `session_notes` - User notes per session
- `global_tool_templates` / `project_tool_templates` - Reusable tool configurations

## Styling

Dark mode only using Tailwind CSS. Key colors:
- Background: `bg-gray-900` (primary), `bg-gray-800` (cards)
- Text: `text-gray-100` (primary), `text-gray-400` (secondary)
- Accent: `text-cyan-400` (links), `text-emerald-400` (success), `text-amber-400` (warning), `text-red-400` (error)

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `5000` | Server port |
| `DB_PATH` | `~/.circuschief/circuschief.db` | SQLite database path |
| `VITE_API_URL` | `http://localhost:5000` | Backend API URL (frontend) |

## Working Directory Guidelines

**CRITICAL: Never use `cd` to change to a hardcoded project path before running commands.**

When running in a Circus Chief session, your working directory is already set correctly. This may be:
- The main project directory
- A git worktree for branch isolation

### Do NOT do this:
```bash
# BAD - bypasses worktree isolation
cd /home/ubuntu/workspace/circus-chief && git status
cd /home/ubuntu/workspace/circus-chief && yarn test
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ferrislucas/Circus-Chief](https://github.com/ferrislucas/Circus-Chief) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Multi-LLM chat interface with Vue.js frontend, Go backend, and PostgreSQL database. Supports OpenAI, Claude, Gemini, and Ollama models with features like workspaces, snapshots, and file uploads.

## Development Commands

### Backend (Go)
```bash
cd api

# Install dependencies and hot-reload tool
make install

# Run server with hot reload (uses Air)
make serve

# Build
make build

# Format code
make fmt

# Run tests
go test ./...

# Regenerate SQLC code (after modifying queries or schema)
sqlc generate
```

**Important**: The backend uses Air for hot-reloading during development. Configuration is in `api/.air.toml`.

### Frontend (Vue.js)
```bash
cd web

# Install dependencies
npm install

# Development server with hot reload
npm run dev

# Build for production
npm run build

# Run linter
npm run lint

# Fix linting issues
npm run lint:fix

# Run tests
npm test
```

### E2E Tests (Playwright)
```bash
cd e2e

# Run all tests
npx playwright test

# Run with UI
npx playwright test --ui
```

## Architecture

### Request Flow
```
HTTP Request → Mux Router → Handler → Service → SQLC Queries → PostgreSQL
                                ↓
                         LLM Provider (OpenAI/Claude/Gemini/Ollama)
```

### Backend Architecture (Go)

**Key Pattern**: The backend follows a handler → service → repository (SQLC) pattern:

1. **Handlers** (`*_handler.go`): HTTP request/response handling
   - `chat_main_handler.go`: Core chat functionality
   - `chat_session_handler.go`: Session CRUD operations
   - `chat_message_handler.go`: Message operations
   - `chat_workspace_handler.go`: Workspace management
   - `chat_auth_user_handler.go`: Authentication
   - `admin_handler.go`: Admin operations

2. **Services** (`*_service.go`): Business logic layer
   - `chat_main_service.go`: Chat orchestration and LLM routing

3. **SQLC Generated Code** (`sqlc_queries/`): Type-safe database queries
   - Schema: `api/sqlc/schema.sql`
   - Queries: `api/sqlc/queries/*.sql`
   - Generated Go: `api/sqlc_queries/*.go`
   - Config: `api/sqlc.yaml`

4. **LLM Integrations** (`llm/`):
   - `llm/openai/`: OpenAI API client
   - `llm/claude/`: Claude API client
   - `llm/gemini/`: Gemini API client
   - Each provider has its own request/response formatting

**Router**: Uses Gorilla Mux for routing (configured in `main.go`)

### Frontend Architecture (Vue.js)

**Stack**: Vue 3 (Composition API) + Pinia + Naive UI + Rsbuild + Tailwind CSS

**Key Directories**:
- `web/src/views/`: Page components
- `web/src/components/`: Reusable components
- `web/src/store/modules/`: Pinia stores for state management
- `web/src/api/`: API client functions
- `web/src/views/chat/composables/`: Chat feature composables (refactored from monolithic component)

**Chat Composables Pattern**: The main chat interface uses a composable-based architecture for better separation of concerns:
- `useStreamHandling.ts`: Handles LLM streaming responses
- `useConversationFlow.ts`: Manages conversation lifecycle
- `useRegenerate.ts`: Message regeneration
- `useSearchAndPrompts.ts`: Search and prompt templates
- `useChatActions.ts`: Snapshot, bot creation, file uploads
- `useErrorHandling.ts`: Centralized error management
- `useValidation.ts`: Input validation rules
- `usePerformanceOptimizations.ts`: Debouncing, memoization

This pattern reduced the main component from 738 to 293 lines while adding better error handling and type safety.

### Database (PostgreSQL + SQLC)

**SQLC Workflow**:
1. Define schema in `api/sqlc/schema.sql`
2. Write SQL queries in `api/sqlc/queries/*.sql`
3. Run `sqlc generate` to create type-safe Go code
4. Use generated code in services

**Key Tables**:
- `auth_user`: User accounts (first registered user becomes admin)
- `chat_session`: Chat sessions
- `chat_message`: Messages within sessions
- `chat_workspace`: Workspace organization
- `chat_model`: Available LLM models
- `chat_prompt`: Prompt templates
- `chat_snapshot`: Shareable conversation snapshots
- `chat_file`: File uploads

**Default Context**: Latest 4 messages are included in context by default.

## Environment Variables

Required variables (set in shell or `.env`):
```bash
# Database (required)
DATABASE_URL=postgres://user:pass@host:port/dbname?sslmode=disable

# LLM API Keys (at least one required)
OPENAI_API_KEY=sk-...
CLAUDE_API_KEY=...
GEMINI_API_KEY=...
DEEPSEEK_API_KEY=...

# Optional
OPENAI_RATELIMIT=100  # Calls per 10 minutes (default: 100)
JWT_SECRET=...         # For JWT token signing
```

**Note**: The "debug" model doesn't require API keys for testing.

## Key Features & Patterns

### Authentication & Authorization
- JWT-based authentication
- First registered user becomes administrator (`is_superuser=true`)
- Rate limiting per user (default: 100 calls/10min, configurable via `OPENAI_RATELIMIT`)
- Per-model rate limiting available for specific models (GPT-4, etc.)

### Workspaces
- Sessions are organized into workspaces
- Each user has a default "General" workspace
- Custom colors and icons for visual organization
- Workspace-specific session isolation

### Chat Flow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swuecho/chat](https://github.com/swuecho/chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->

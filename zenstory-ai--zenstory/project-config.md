---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to coding agents when working with code in this repository.

## Project Overview

**zenstory** is an AI-assisted novel writing workbench with a conversational AI interface. Files (outlines, drafts, characters, lores) are the central unit, with AI conversations driving content generation.

**Architecture**: Monorepo with React frontend (apps/web) and FastAPI backend (apps/server). Three-panel layout: File tree (left), Editor (middle), AI Chat (right).

**Related Documentation**:
- `apps/server/agent/CLAUDE.md` - Detailed Agent system architecture (multi-agent workflow, tools, SSE events)

## Development Commands

### Backend (FastAPI/Python)

```bash
cd apps/server

# Install dependencies (first time only)
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Start backend server (port 8000)
python3 main.py

# Run specific Alembic migration
alembic upgrade head

# Create new migration
alembic revision --autogenerate -m "description"
```

### Frontend (React/Vite)

```bash
cd apps/web

# Install dependencies
pnpm install  # or npm install

# Start development server (port 5173)
pnpm dev  # or npm run dev

# Build for production
pnpm build

# Type-check + build
pnpm run build:typecheck

# Lint
pnpm lint
```

### Running Both Services

From project root (requires two terminals):
```bash
# Terminal 1: Backend
cd apps/server && source venv/bin/activate && python3 main.py

# Terminal 2: Frontend
cd apps/web && pnpm dev
```

### Database Migrations

The backend uses Alembic for database migrations:
- Migrations are in `apps/server/alembic/versions/`
- Configuration: `apps/server/alembic.ini`
- After schema changes: `alembic revision --autogenerate -m "description"` then `alembic upgrade head`

## Architecture Overview

### Backend (apps/server)

**Tech Stack**: FastAPI, SQLModel (Pydantic + SQLAlchemy), SQLite (dev) / PostgreSQL (production), DeepSeek OpenAI-compatible API with openai-agents-python, LlamaIndex, ChromaDB

**Key Directories**:
- `main.py` - App entry point, registers all routers and middleware
- `api/` - API route handlers (auth, projects, files, versions, agent, chat, export, voice)
- `models/` - SQLModel database models (entities, file_model, file_version)
- `services/` - Business logic layer (file_version_service, snapshot_service, verification_service, export_service)
- `agent/` - AI Agent system (service.py, suggest_service.py, tools/, context/, schemas/)
- `database.py` - Database connection and session management
- `config/` - Configuration modules (logger, settings)

**Important**: The codebase uses a service layer pattern. Route handlers in `api/` should be thin and delegate business logic to `services/`.

### Frontend (apps/web)

**Tech Stack**: React 19, TypeScript, Vite, Tailwind CSS 4.x, Zustand, TanStack React Query, Tiptap editor, react-arborist (file tree)

**Key Directories**:
- `components/` - React components (Layout, FileTree, Editor, ChatPanel, MessageList, etc.)
- `contexts/` - React Context providers (AuthContext, ProjectContext, ThemeContext)
- `hooks/` - Custom hooks (useAgentStream, useVoiceInput, useExport)
- `lib/` - Utilities (api.ts, apiClient.ts, agentApi.ts, errorHandler.ts)
- `types/` - TypeScript type definitions

**State Management**:
- `AuthContext` - User authentication state
- `ProjectContext` - Active project and file state
- `Zustand` stores - Additional state management
- `TanStack Query` - Server state caching and synchronization

### Agent System

The AI Agent system is the core feature:

**Backend (apps/server/agent/)**:
- `service.py` - Main agent orchestration with streaming response
- `suggest_service.py` - AI-powered content suggestions
- `context/` - Context assembly and prioritization for RAG
- `tools/` - Agent tools (file_executor.py for file operations)
- `schemas/` - Request/response models
- `prompts/` - System prompts for different agent behaviors

**Frontend**:
- `useAgentStream.ts` - SSE (Server-Sent Events) streaming hook
- `ChatPanel.tsx` - Main AI chat interface
- `MessageList.tsx` - Message rendering with tool results
- `ToolResultCard.tsx` - Display tool execution results

### File Search

Global file search functionality for quickly finding files within the current project.

**Components**:
- `FileSearchInput` - Search input with debounce and IME support
- `SearchResultsDropdown` - Keyboard-navigable results dropdown
- `FileTypeFilter` - File type filtering dropdown

**Hook**: `useFileSearch`
- Client-side search with fuzzy matching
- 300ms debounce
- Max 50 results
- Case-insensitive

**Context**: `FileSearchContext`
- Global search state management
- Keyboard shortcut support

**Trigger**: Cmd+K (Mac) / Ctrl+K (Windows/Linux)

**Scope**: Current project files

**Matching**: Fuzzy, case-insensitive title search with relevance ranking (exact > prefix > contains)

**Features**:
- Real-time search with debouncing
- File type filtering (outline, draft, character, lore, material)
- Keyboard navigation (ArrowUp/Down, Enter, Escape)
- Global keyboard shortcut
- Mobile-responsive design
- i18n support (EN/ZH)
- IME composition support for Chinese input

## Key Configuration Files

### Backend Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zenstory-ai/zenstory](https://github.com/zenstory-ai/zenstory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->

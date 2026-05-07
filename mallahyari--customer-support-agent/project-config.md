---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chirp is an open-source, self-hostable AI chatbot widget that enables businesses to add AI-powered customer support to their websites. The system creates a knowledge base from provided content (URL scraping or direct text input) and uses RAG (Retrieval-Augmented Generation) to answer visitor questions.

**Key Architecture:**
- **Backend:** FastAPI + SQLAlchemy (async) + SQLite
- **Vector Store:** Qdrant (local/embedded mode or server)
- **AI:** OpenAI API (embeddings: text-embedding-3-small, chat: gpt-4o-mini)
- **Frontend Dashboard:** React + Vite + TypeScript + Tailwind CSS + Shadcn/UI
- **Widget:** React compiled to single widget.js via Vite library mode
- **Auth:** Session-based (bcrypt passwords, session tokens)

## Development Commands

### Backend

The backend uses Python with uv for package management (configured in `pyproject.toml`).

```bash
# Navigate to backend directory
cd backend

# Install dependencies (using uv)
uv pip install -r requirements.txt

# Run development server
uvicorn main:app --reload --port 8000

# Run with specific host
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

**Environment Setup:**
- Copy `.env` file with required API keys (see `backend/.env` for structure)
- Required: `OPENAI_API_KEY`, `GOOGLE_API_KEY`, admin credentials
- Database URL, ChromaDB path, and upload paths are configured via environment variables

### Frontend

The frontend is a React application using Vite and Tailwind CSS v4.

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Run development server (default port 5173)
npm run dev

# Build for production
npm run build

# Type checking
tsc -b

# Lint
npm run lint

# Preview production build
npm run preview
```

**Important Frontend Details:**
- Uses `@` path alias pointing to `./src` (configured in vite.config.ts)
- Shadcn/UI components are in `src/components/ui/`
- Tailwind CSS v4 with `@tailwindcss/vite` plugin
- Component configuration in `components.json`

### Widget Build

The embeddable widget is built separately from the dashboard:

```bash
cd widget

# Install dependencies
npm install

# Build widget (outputs widget.js)
npm run build

# Development mode with hot reload
npm run dev
```

The widget uses Vite library mode to compile to a single `widget.js` file with Shadow DOM isolation.

## Project Structure

### Backend (`/backend`)

```
backend/
├── app/                 # Currently empty, backend code needs implementation
├── .env                 # Environment variables (NOT committed - contains API keys)
└── requirements.txt     # Python dependencies
```

**Key Backend Components (to be implemented per docs):**
- `main.py` - FastAPI app entry point
- `database.py` - SQLAlchemy setup (async SQLite)
- `models.py` - DB models (Bot, Conversation, Message, AdminSession)
- `schemas.py` - Pydantic models for validation
- `auth.py` - Authentication utilities
- `routes/` - API endpoints (auth, admin, public, chat)
- `services/` - Business logic (scraper, embeddings, chat_service)

### Frontend (`/frontend`)

```
frontend/
├── src/
│   ├── App.tsx              # Main app component
│   ├── main.tsx             # Entry point
│   ├── components/
│   │   ├── ui/              # Shadcn/UI components (Base UI + Radix)
│   │   ├── component-example.tsx
│   │   └── example.tsx
│   ├── lib/
│   │   └── utils.ts         # Utility functions (cn, etc.)
│   └── index.css            # Global styles + Tailwind
├── vite.config.ts           # Vite configuration
├── tsconfig.json            # TypeScript config
└── components.json          # Shadcn/UI configuration
```

**UI Component Library:**
- Uses Base UI (headless React components from MUI team)
- Styled with Tailwind CSS v4 and class-variance-authority
- Components: AlertDialog, Badge, Button, Card, Combobox, DropdownMenu, Field, Input, InputGroup, Label, Select, Separator, Textarea

### Documentation (`/docs`)

Comprehensive documentation exists in the `docs/` folder:
- `QUICK-START.md` - Tech stack, database schema, folder structure, implementation steps
- `API-SPEC.md` - Complete API endpoint specifications with examples
- `PRD-CORE.md` - Full product requirements document with architecture decisions

**Always reference these docs when:**
- Implementing new features
- Understanding expected API contracts
- Questions about database schema or architecture
- Understanding business rules and constraints

## Database Schema

**SQLite with SQLAlchemy (async):**

### Tables

**bots:**
- `id` (TEXT PK) - UUID
- `name`, `welcome_message`, `avatar_url`
- `accent_color` (default: #3B82F6)
- `position` (bottom-right|bottom-left|bottom-center)
- `show_button_text` (BOOLEAN), `button_text`
- `source_type` (url|text), `source_content`
- `api_key` (UUID, unique)
- `message_count`, `message_limit` (default: 1000)
- `created_at`, `updated_at`

**conversations:**
- `id` (TEXT PK), `bot_id` (FK)
- `session_id` - Browser fingerprint from widget
- `created_at`, `updated_at`

**messages:**
- `id` (TEXT PK), `conversation_id` (FK)
- `role` (user|assistant), `content`
- `created_at`

**admin_sessions:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mallahyari/customer-support-agent](https://github.com/mallahyari/customer-support-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

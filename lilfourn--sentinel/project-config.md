---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sentinel** is a Tauri v2 desktop file manager with AI-powered organization. React 19 frontend + Rust backend + Convex cloud backend, featuring agentic AI that explores folders and creates organization plans. Users pay for a subscription (free/pro tiers) to access AI features - we provide the API keys.

### Core Features
- File browsing (grid/list/column views) with drag-and-drop
- **AI Chat** - Conversational file exploration with Claude (streaming + extended thinking)
- **Agentic Organization** - Autonomous folder analysis and reorganization
- **Virtual File System (VFS)** - Preview changes before applying
- **Write-Ahead Log (WAL)** - Crash recovery and transactional safety
- **Vector Search** - Semantic file search via local embeddings

## Commands

```bash
# Development
npm run tauri dev          # Development (Vite + Tauri)
npm run tauri build        # Production build

# Frontend
npm run build              # Type check (tsc && vite build)
npm test                   # Run frontend tests (vitest watch)
npm run test:run           # Run frontend tests once
npm run test:coverage      # Run tests with coverage
npm test -- src/stores/chat-store.test.ts  # Run single test file

# Rust backend (from src-tauri/)
cargo check                # Type checking
cargo test                 # Run all tests
cargo test test_name       # Run single test
cargo build --profile release-fast  # Fast release build for testing

# Convex (cloud backend)
npx convex dev             # Start Convex dev server
npx convex deploy          # Deploy to production
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend (React 19)                       │
├──────────────────┬──────────────────┬───────────────────────┤
│ ChatPanel        │ Stores (Zustand) │ Views                 │
│ - Streaming      │ - chat-store     │ - FileGridView        │
│ - Tool viz       │ - organize-store │ - FileListView        │
│ - @mentions      │ - vfs-store      │ - FileColumnsView     │
│                  │ - ghost-store    │ - DiffView            │
│ ChangesPanel     │ - navigation     │                       │
│ - Plan preview   │ - selection      │ Dialogs               │
│ - Execution      │ - settings       │ - ConfirmDialog       │
│ - Ghost overlay  │                  │ - ContextMenu         │
└──────────────────┴──────────────────┴───────────────────────┘
        │ invoke()                              │ Convex client
        ▼                                       ▼
┌───────────────────────────────┐    ┌────────────────────────┐
│   Backend (Rust + Tauri)      │    │   Convex Cloud         │
├───────────────────────────────┤    ├────────────────────────┤
│ Commands                      │    │ users, userSettings    │
│ - chat, ai, filesystem        │    │ subscriptions          │
│ - vfs, wal, jobs              │    │ dailyUsage, history    │
│                               │    │ stripeWebhookEvents    │
│ AI Module                     │    └────────────────────────┘
│ - chat/agent (ReAct)          │
│ - v2/agent_loop (organize)    │
│ - grok/ (OpenAI workers)      │
│ - rules/ (DSL parser)         │
│                               │
│ Infrastructure                │
│ - vfs/ (simulation)           │
│ - wal/ (journaling)           │
│ - execution/ (DAG)            │
│ - vector/ (embeddings)        │
│ - billing/ (limits)           │
└───────────────────────────────┘
        │ HTTP
        ▼
┌─────────────────────────────────────────────────────────────┐
│                    AI APIs                                   │
│  Claude (Haiku/Sonnet)       │  OpenAI (GPT-5-nano/mini)    │
└─────────────────────────────────────────────────────────────┘
```

## Frontend (src/)

**Stack**: React 19, TypeScript, Vite 7, TailwindCSS v4, Zustand, TanStack Query, Clerk (auth)

### Key Stores
| Store | Purpose |
|-------|---------|
| `chat-store.ts` | Chat messages, streaming, context items, model selection |
| `organize-store.ts` | Organization workflow state machine, plan execution |
| `vfs-store.ts` | Virtual filesystem preview state |
| `ghost-store.ts` | Ghost animations for file operation previews |
| `navigation-store.ts` | Directory navigation, history, view mode |
| `selection-store.ts` | Multi-file selection |
| `subscription-store.ts` | Billing tier, usage limits |

### Chat System (`src/components/ChatPanel/`)
- **ChatInput** - Model selector (Haiku/Sonnet), @mention support
- **MessageItem** - Extended thinking display, tool visualization (memoized)
- **ThoughtAccordion** - Shows agent tool calls (search, read, inspect)
- **InlineMentionDropdown** - File/folder picker with context strategies

## Backend (src-tauri/)

### Module Structure
```
src-tauri/src/
├── commands/        # Tauri command handlers
├── ai/
│   ├── chat/        # ReAct agent for Q&A (streaming, extended thinking)
│   ├── v2/          # Organize agent (map-reduce, hologram compression)
│   ├── grok/        # OpenAI integration (explore agent, vision, document parser)
│   └── rules/       # DSL rule parser and evaluator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lilfourn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

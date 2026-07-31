---
trigger: always_on
description: This is your navigation hub. Start here, follow the links, and return when you need orientation.
---

# MiniSearch Agent Guidelines

This is your navigation hub. Start here, follow the links, and return when you need orientation.

## Before You Start

**New to this codebase?** Read in this order:
1. `docs/quick-start.md` - Get it running
2. `docs/overview.md` - Understand the system
3. `docs/project-structure.md` - Navigate the code

**Making changes?** Check:
- `docs/coding-conventions.md` - Code style
- `docs/development-commands.md` - Available commands
- `docs/pull-requests.md` - How to submit

## Repository Map

### Getting Started
- **`docs/quick-start.md`** - Installation, first run, verification
- **`docs/overview.md`** - System architecture and data flow
- **`docs/project-structure.md`** - Directory layout and component organization

### Configuration & Setup
- **`docs/configuration.md`** - Environment variables and settings reference
- **`docs/security.md`** - Access control, privacy, and security model

### Core Functionality
- **`docs/ai-integration.md`** - AI inference types (Wllama, OpenAI, AI Horde, Internal)
- **`docs/ui-components.md`** - Component architecture and PubSub patterns
- **`docs/search-history.md`** - History database schema and management
- **`docs/conversation-memory.md`** - Token budgeting and rolling summaries
- **`docs/reranking.md`** - Reranker subsystem and llama-server lifecycle
- **`docs/glossary.md`** - Codebase-specific terms and domain concepts

### Development
- **`docs/development-commands.md`** - Docker, npm, and testing commands
- **`docs/coding-conventions.md`** - Style guide and patterns
- **`docs/pull-requests.md`** - PR process and merge philosophy
- **`docs/core-technologies.md`** - Technology stack and dependencies
- **`docs/design.md`** - UI/UX design principles

## Agent Decision Tree

```
Need to:
├── Add a feature?
│   ├── UI component → docs/ui-components.md
│   ├── AI integration → docs/ai-integration.md
│   ├── Search functionality → client/modules/search.ts
│   └── Settings option → docs/configuration.md
├── Fix a bug?
│   ├── UI issue → Check component + PubSub channels
│   ├── AI not working → docs/ai-integration.md + browser console
│   ├── Search failing → Check SearXNG + server hooks
│   └── Build error → docs/development-commands.md
├── Configure deployment?
│   ├── Environment variables → docs/configuration.md
│   ├── Access control → docs/security.md
│   └── Docker setup → docs/overview.md
└── Understand data flow?
    ├── Search flow → client/modules/search.ts
    ├── AI generation → client/modules/textGeneration.ts
    ├── State management → docs/ui-components.md
    └── History/Chat → docs/search-history.md + docs/conversation-memory.md
```

## Key Files Reference

### Entry Points
- `client/index.tsx` - React app initialization
- `vite.config.ts` - Vite dev server with hooks
- `Dockerfile` - Multi-stage container build

### Business Logic Modules
- `client/modules/search.ts` - Search orchestration and caching
- `client/modules/textGeneration.ts` - AI response flow
- `client/modules/pubSub.ts` - All PubSub channels
- `client/modules/settings.ts` - Settings management
- `client/modules/history.ts` - Search history database

### Server-Side Modules
- `server/searchEndpointServerHook.ts` - `/search` endpoints
- `server/internalApiEndpointServerHook.ts` - `/inference` proxy
- `server/webSearchService.ts` - SearXNG integration
- `server/rerankerService.ts` - Local result reranking

### UI Components
- `client/components/App/` - Application shell with error boundaries
- `client/components/Search/Form/` - Search input and form
- `client/components/Search/Results/` - Textual and graphical results display
- `client/components/Search/History/` - History drawer and button
- `client/components/AiResponse/` - AI response display and chat interface
- `client/components/Pages/Main/` - Main page layout
- `client/components/Pages/Main/Menu/` - Settings drawers (AI, Search, Interface, History, Voice, Actions)
- `client/components/Pages/AccessPage.tsx` - Access key validation page
- `client/components/Analytics/SearchStats.tsx` - Search analytics cards
- `client/components/Logs/` - Application logging modal
- `client/components/Settings/HistorySettings.tsx` - History configuration UI

### Client Modules
- `client/modules/pubSub.ts` - All PubSub channels for state management
- `client/modules/search.ts` - Search orchestration and IndexedDB caching
- `client/modules/textGeneration.ts` - AI response generation and chat handling
- `client/modules/textGenerationWithWllama.ts` - Browser-based inference
- `client/modules/textGenerationWithOpenAi.ts` - OpenAI-compatible API inference
- `client/modules/textGenerationWithHorde.ts` - AI Horde distributed inference
- `client/modules/textGenerationWithInternalApi.ts` - Internal API proxy inference
- `client/modules/settings.ts` - Default settings and inference type definitions
- `client/modules/history.ts` - IndexedDB persistence for searches and chat
- `client/modules/wllama.ts` - Wllama model configuration and management
- `client/modules/webGpu.ts` - WebGPU availability detection (`"gpu" in navigator`)
- `client/modules/querySuggestions.ts` - Search suggestion UI, stored in IndexedDB
- `client/modules/relatedSearchQuery.ts` - Generates related search queries

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felladrin/MiniSearch](https://github.com/felladrin/MiniSearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI-powered map control framework using `@carto/agentic-deckgl`. Users interact with a deck.gl map through natural language chat. Messages are processed by an LLM (via multiple AI SDK backends) that generates tool calls executed client-side to manipulate the map.

**Tech Stack:**
- Core Library: `@carto/agentic-deckgl` (TypeScript, Zod, framework-agnostic)
- Backends: Node.js + TypeScript, Express, WebSocket — OpenAI Agents SDK (default), Vercel AI SDK v6, or Google ADK
- Frontends: Angular 20, React 19, Vue 3, Vanilla JS — all with deck.gl, MapLibre GL, CARTO

## Project Structure

```
carto-agentic-deckgl/                        # Root IS the library package
├── src/                                     # Core library source (@carto/agentic-deckgl)
├── test/                                    # Core library tests
├── dist/                                    # Build output (ESM + CJS)
├── docs/                                    # Centralized documentation guides
├── examples/
│   ├── backend/
│   │   ├── openai-agents-sdk/               # Backend server — OpenAI Agents SDK (default)
│   │   ├── vercel-ai-sdk/                   # Backend server — Vercel AI SDK
│   │   └── google-adk/                      # Backend server — Google ADK
│   └── frontend/
│       ├── angular/                         # Angular 20 frontend
│       ├── react/                           # React 19 frontend (+ E2E tests)
│       ├── vue/                             # Vue 3 frontend
│       └── vanilla/                         # Vanilla JS frontend
├── package.json                             # Library package.json
├── rollup.config.js                         # Build config
├── tsconfig.json                            # TypeScript config
└── vitest.config.ts                         # Test config
```

## Development Commands

### Core Library
```bash
npm install
npm run build        # Build ESM + CJS outputs to dist/
npm run dev          # Watch mode
npm run type-check   # Type check without emitting
npm test             # Run unit tests
```

### Backend (OpenAI Agents SDK — default)
```bash
cd examples/backend/openai-agents-sdk
npm run dev          # Start dev server with hot reload (tsx watch, port 3003)
npm run dev:mock-mcp # Start with MCP mock mode (fixture-backed tools)
npm run build        # Compile TypeScript to dist/
npm start            # Run production build
npm run typecheck    # Type check without emitting
```

### Backend (Vercel AI SDK)
```bash
cd examples/backend/vercel-ai-sdk
npm run dev          # Start dev server with hot reload (tsx watch, port 3003)
npm run build        # Compile TypeScript to dist/
npm start            # Run production build
npm run typecheck    # Type check without emitting
```

### Backend (Google ADK)
```bash
cd examples/backend/google-adk
npm install --force  # --force needed for peer dependency conflicts
npm run dev          # Start dev server with hot reload (tsx watch, port 3003)
npm run build        # Compile TypeScript to dist/
npm start            # Run production build
npm run typecheck    # Type check without emitting
```

### Frontend (Angular)
```bash
cd examples/frontend/angular
pnpm install         # Install dependencies
pnpm start           # Start dev server (http://localhost:4200)
pnpm build           # Build for production
```

### Frontend (React)
```bash
cd examples/frontend/react
pnpm install         # Install dependencies
pnpm dev             # Start dev server (http://localhost:5173)
pnpm build           # Build for production
pnpm test            # Run unit tests
```

### E2E Tests (React)
```bash
cd examples/frontend/react
npx playwright install chromium                     # One-time browser install
pnpm e2e                                            # Headless (default: openai-agents-sdk backend)
pnpm e2e:headed                                     # Headed mode (watch in browser)
pnpm e2e:ui                                         # Interactive UI mode
pnpm e2e -- --grep "Counties"                       # Run a single test by name
BACKEND_SDK=vercel-ai-sdk pnpm e2e                  # Run against Vercel AI SDK backend
pnpm e2e:update-snapshots                           # Update screenshot baselines
pnpm e2e:report                                     # View HTML report
pnpm e2e:matrix                                     # Run full model matrix (default backend)
pnpm e2e:matrix --backend vercel-ai-sdk             # Run matrix against Vercel backend
pnpm e2e:matrix --backend openai-agents-sdk --current  # Run matrix with current model
```

### Running the Application

1. Start backend: `cd examples/backend/openai-agents-sdk && npm run dev` (runs on http://localhost:3003)
2. Start frontend: `cd examples/frontend/angular && pnpm start` (http://localhost:4200) or `cd examples/frontend/react && pnpm dev` (http://localhost:5173)

## Architecture

### Communication Flow

```mermaid
flowchart TD
    A[User Message] --> B[Frontend WebSocket]
    B --> C[Backend - Express]
    C --> D[AI SDK\nstreaming + tool calling]
    D --> E[text chunks + tool_call messages]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CartoDB/carto-agentic-deckgl](https://github.com/CartoDB/carto-agentic-deckgl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->

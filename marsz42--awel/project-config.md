---
trigger: always_on
description: Awel is an AI-powered development overlay for Next.js applications. It runs a proxy server that sits in front of the user's dev server, injects a floating button into the page, and opens an interactive dashboard (in an iframe) where users can chat with an AI agent that can read, write, and edit files in the project.
---

# CLAUDE.md — Awel Codebase Guide

## What is Awel?

Awel is an AI-powered development overlay for Next.js applications. It runs a proxy server that sits in front of the user's dev server, injects a floating button into the page, and opens an interactive dashboard (in an iframe) where users can chat with an AI agent that can read, write, and edit files in the project.

## Project Structure

A single npm package (`awel`) with source code organized under `packages/`. Uses ES modules (`"type": "module"`) throughout. Published as a single package with `bin: { awel: "./bin/awel.js" }`.

```
packages/
  cli/         – Hono server, agent orchestration, LLM integration, proxy
  dashboard/   – React UI for chatting with the agent (embedded iframe)
  host/        – Vanilla JS script injected into the user's app (Shadow DOM)
```

Output structure after build:

```
dist/
  cli/         – Compiled CLI JS + skills/
  dashboard/   – Vite-built React SPA
  host/        – Single bundled host.js (IIFE)
bin/
  awel.js      – CLI entry point
```

## Build & Dev Commands

```bash
npm run build              # Build all packages (host → dashboard → cli)
npm run build:cli          # tsc + copy skills/ to dist
npm run build:host         # esbuild → dist/host/host.js (IIFE, minified)
npm run build:dashboard    # Vite build → dist/dashboard/
npm run dev                # CLI watch mode (tsc --watch)
npm run test               # vitest run
npm run test:watch         # vitest watch

# Create a new project with Awel (scaffolds Next.js + marks for creation mode)
npx awel create

# Running Awel against a Next.js app
npx awel dev               # Start on default ports (Awel:3001, app:3000)
npx awel dev -p 4000       # Target app on port 4000
npx awel dev -v            # Verbose mode (prints LLM stream events to stderr)
```

## Package Details

### CLI (`packages/cli/`)

The core server and agent orchestration layer.

- **Runtime**: Node.js, TypeScript compiled with `tsc`
- **Web framework**: Hono (served via `@hono/node-server`)
- **AI SDK**: Vercel AI SDK v6 (`ai` package) as the unified LLM abstraction
- **Ports**: Awel server on 3001 (`AWEL_PORT`), proxies to user app on 3000 (`USER_APP_PORT`). Configured in `src/config.ts`.

**Key source files:**

| File | Purpose |
|------|---------|
| `src/index.ts` | CLI entry point (Commander.js). Defines `awel dev` and `awel create` commands. |
| `src/server.ts` | Hono app setup: mounts API routes, serves dashboard/host static files, proxies everything else to user's app. Handles WebSocket upgrades for HMR. Manages creation mode state and project status endpoints. |
| `src/agent.ts` | Agent API routes: `POST /api/stream` (SSE streaming), `GET /api/history`, `GET /api/models`, `DELETE /api/chat/history`, plan/question/confirmation approval endpoints. |
| `src/session.ts` | Multi-turn conversation state. Model-aware session caching and message history preservation across model switches. Persists to `.awel/session.json`. |
| `src/config.ts` | Port defaults (`AWEL_PORT`, `USER_APP_PORT`), MIME type mappings. |
| `src/types.ts` | Shared type definitions. |
| `src/providers/registry.ts` | Model catalog and provider resolution. Maps model IDs → providers. |
| `src/providers/vercel.ts` | Core streaming implementation using Vercel AI SDK `streamText()`. Handles tool execution, SSE event emission, chat history management. Uses a specialized creation system prompt when in creation mode. |
| `src/providers/types.ts` | Shared types: `StreamProvider`, `ModelDefinition`, `ProviderConfig` (includes `creationMode` flag). |
| `src/proxy.ts` | HTTP proxy middleware. Intercepts HTML responses to inject the host script (`/_awel/host.js`). In creation mode, serves the dashboard as a full-page app at `/` instead of proxying. |
| `src/subprocess.ts` | Dev server process management: spawning via execa, health checks, auto-restart on crash, status tracking. |
| `src/devserver.ts` | HMR WebSocket traffic pause/resume during agent streams. |
| `src/undo.ts` | Session-based undo system. Captures git baseline, tracks changed files, stack-based LIFO rollback. |
| `src/plan-store.ts` | Singleton store for proposed plans awaiting user approval. |
| `src/confirm-store.ts` | Request/resolve confirmation for destructive tool operations (bash, file writes). Separate auto-approve flags for `bash` and `fileWrites`. |
| `src/memory.ts` | Persistent project knowledge system. Stores memories in `.awel/memory.json` with recency/usage scoring. Two scopes: "always" (injected every request) and "contextual" (searched on-demand). |
| `src/sse.ts` | SSE event helper utilities. Chat history storage and persistence to `.awel/history.json`. |
| `src/inspector.ts` | Inspector relay routes for element selection. |
| `src/babel-setup.ts` | Babel plugin setup for source mapping. |
| `src/comment-popup.ts` | Comment popup handling. |
| `src/logger.ts` | Logging utilities. |
| `src/verbose.ts` | Verbose mode tracking. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarsZ42/Awel](https://github.com/MarsZ42/Awel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->

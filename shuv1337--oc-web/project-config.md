---
trigger: always_on
description: This file provides context and guidelines for GitHub Copilot coding agent when working on the OC Web project. These instructions help ensure consistency, quality, and alignment with project conventions.
---

# GitHub Copilot Instructions for OC Web

This file provides context and guidelines for GitHub Copilot coding agent when working on the OC Web project. These instructions help ensure consistency, quality, and alignment with project conventions.

## Project Overview

OC Web is a web-based interface for the OpenCode Server API, providing a browser-based way to interact with OpenCode sessions. Built on TanStack Start, React, and Bun, it offers a complete web experience for managing and monitoring OpenCode workflows.

## Technology Stack

- **Runtime**: Bun 1.3.x (toolchain pinned in `bunfig.toml`)
- **Framework**: TanStack Start + React Router (hybrid SSR/CSR routing)
- **UI Library**: React 19.1.0
- **Language**: TypeScript 5.x
- **Styling**: Tailwind CSS 4.x
- **Build Tool**: Vite 6.x
- **Server**: Bun server (`server.ts`) that proxies event streams to OpenCode backend

## Build/Test Commands

- **Install dependencies**: `bun install`
- **Development server**: `bun run dev` (launches with hot reload on http://localhost:3000)
- **Production build**: `bun run build` (outputs to `dist/client` and `dist/server`)
- **Production server**: `bun run start` (runs `server.ts` with production bundles)
- **Linting**: `bun run lint` (enforces shared ESLint rules)
- **Type checking**: `bun x tsc --noEmit` (typecheck without generating artifacts)
- **Format check**: `bun x prettier --check .`
- **Tests**: `bun run test` (runs Playwright smoke tests when present)

## Code Style & Conventions

### General Guidelines

- **Avoid `any` type**: Use proper TypeScript types and interfaces
- **Minimize `let`**: Prefer `const` for immutable bindings
- **No unnecessary destructuring**: Only destructure when it improves readability
- **Avoid `else` blocks**: Use early returns and guard clauses for cleaner code
- **Keep functions focused**: Extract helpers only when they're reused across multiple components
- **Bun APIs**: Prefer Bun utilities (`Bun.file`, `Bun.readableStreamFromAsyncIterator`, etc.) in shared utilities when they simplify IO or streaming

### TypeScript

- Use TypeScript interfaces/types for data structures
- Use Zod schemas when validating server function inputs
- Keep server function schemas synced with the OpenCode SDK

### Logging

- Guard all logging behind development checks: `if (process.env.NODE_ENV !== "production")`
- Never log secrets, API keys, or sensitive data
- Use console.log sparingly; consider the `no-console-log` rule in `rules/no-console-log.yml`

### React Components

- Keep components focused on a single responsibility
- Extract reusable UI primitives to `src/app/_components/ui/`
- Follow TanStack Start conventions for file-based routes
- Confirm UI changes work in both desktop and mobile layouts

## Project Structure

```
src/
├── app/                        # TanStack Start routes & UI components
│   └── _components/            # Message renderers, UI primitives, dialogs, pickers
├── contexts/                   # React contexts for session + theme state
├── hooks/                      # Reusable hooks around OpenCode data flows
├── lib/                        # HTTP client, command parser, theme helpers
├── router.tsx                  # Router configuration
server.ts                       # Bun production server wrapper
vite.config.ts                  # Vite + TanStack Start configuration
```

### Key Entry Points

- `src/lib/opencode-server-fns.ts` – server-side wrappers for the OpenCode HTTP API
- `src/app/_components/message/` – renders reasoning, snapshots, tool output, and patches
- `src/app/_components/ui/` – button, dialog, picker, and form controls

## Project Patterns

### OpenCode SDK Integration

- **Reuse OpenCode SDK types** instead of duplicating shapes
- Create local wrappers only when the upstream SDK is unavailable
- When updating server endpoints that proxy OpenCode, note it in PR descriptions for API compatibility tracking
- Browser access to OpenCode server must be mediated through the web app's `/api/*` endpoints (frontend cannot directly connect)

### TanStack Start Conventions

- Follow file-based routing in `src/app/`
- Use server functions for OpenCode HTTP calls
- Keep SSR/CSR boundaries clear

### Environment Variables

Use canonical environment variables:
- `OPENCODE_SERVER_URL`: Full URL to OpenCode server (default: `http://localhost:4096`)
- `OPENCODE_WEB_HOST`: Interface to bind web server to (default: `localhost`)
- `OPENCODE_WEB_PORT`: Port for web UI server (default: `3000`)

Configuration precedence: CLI flags > Canonical env vars > Legacy env vars > Defaults

## Security Considerations

- Never commit secrets or API keys to source code
- Validate all user inputs with Zod schemas in server functions
- Follow security best practices for handling sensitive data
- The application runs without authentication by default - designed for local development only

## Pull Request Checklist

Before submitting a PR:

1. **Format**: Run `bun x prettier --check .`
2. **Lint**: Run `bun run lint` and fix all issues
3. **Type check**: Run `bun x tsc --noEmit` and resolve all errors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuv1337/oc-web](https://github.com/shuv1337/oc-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

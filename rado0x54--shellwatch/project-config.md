---
trigger: always_on
description: This is the canonical project instructions file for Claude Code working in this repo.
---

# ShellWatch — Claude Code Instructions

This is the canonical project instructions file for Claude Code working in this repo.

## Project Overview

ShellWatch is a Human-in-the-Loop platform for agent-driven SSH. It's passkey-first and passkey-only — no passwords anywhere — with an SSH-agent proxy that forwards signing requests end-to-end to a user's WebAuthn passkey. Every agent action surfaces in realtime notifications, persists in a tamper-evident audit log, and can be gated behind explicit human approval before it touches the remote host.

Operationally, it brokers terminal sessions between configured SSH targets, human users (via the web UI), and AI agents (via MCP):

- A browser-based terminal UI for interactive SSH sessions
- An MCP (Model Context Protocol) interface for programmatic session control
- A shared TerminalManager that both UI and MCP operate on

## Tech Stack

- **Runtime:** Node.js with TypeScript (strict mode)
- **Backend:** Fastify with plugins (`@fastify/websocket`, `@fastify/cors`) — owns all server logic (API, WebSocket, MCP, SSH)
- **Frontend:** SvelteKit (adapter-static, client-side SPA) with Svelte 5, xterm.js — routing, layouts, and build only; no SSR or server-side SvelteKit features (Fastify handles that)
- **SSH:** ssh2 library
- **Terminal:** xterm.js
- **MCP:** @modelcontextprotocol/sdk (streamable HTTP transport)
- **Persistence:** Drizzle ORM with SQLite (audit log, pending actions, key/passkey storage)
- **Auth:** WebAuthn passkeys (human login + SSH signing). OAuth2/OIDC is **fully delegated to Ory Hydra** (web UI, MCP, and agent all use mediated DCR + authorization_code + PKCE); ShellWatch is Hydra's passkey-gated login/consent provider and verifies opaque bearer tokens via introspection. No passwords, no API keys.
- **Config:** YAML with zod validation
- **Testing:** Vitest (unit + integration)
- **Linting:** ESLint (typescript-eslint + eslint-plugin-svelte)
- **Formatting:** Prettier (prettier-plugin-svelte)
- **Package manager:** pnpm
- **Agent client:** Go binary in `agent-client/` (separate module, MIT-licensed)

## Code Conventions

- Use ES modules (`import`/`export`), not CommonJS (`require`)
- Destructure imports when possible: `import { foo } from 'bar'`
- TypeScript strict mode — no `any` unless absolutely necessary
- Use `.js` extensions in relative import paths (required for Node16 module resolution)
- Single-line commit messages with category prefix: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`
- Do not add "Generated with Claude Code" or similar AI attribution to commits or PRs
- **Functions with 5+ parameters must use a typed parameter object** instead of positional args. Export the params interface for callers.
- **Every new source file must carry an `SPDX-License-Identifier` header on its first line** (after the shebang, if any). Use `LicenseRef-FSL-1.1-Apache-2.0` for files at the repo root and `MIT` for anything under `agent-client/`. CI enforces this via `pnpm spdx:check`; run `pnpm spdx:write` to add headers to files you've just created. Comment styles by extension: `// …` for `.ts`/`.mjs`/`.js`/`.go`, `<!-- … -->` for `.svelte`/`.html`, `/* … */` for `.css`, `# …` for `.sh`/`Makefile`.

## Project Structure

```
src/
  index.ts              # Entry point — starts Fastify, loads config
  config/               # Config schema (zod) and YAML loader
  server/               # Fastify app, HTTP routes, WebSocket handler
  terminal/             # TerminalManager, OutputBuffer, transport interface
  transport/            # SSH transport implementation (ssh2)
  agent/                # AgentSession — per-agent session isolation
  agent-socket/         # Agent socket transport (Go agent-client bridge)
  mcp/                  # MCP server and streamable HTTP transport
  hydra/                # Ory Hydra integration: passkey login/consent providers, mediated DCR, bearer introspection, discovery
  webauthn/             # WebAuthn passkey registration and authentication
  pending-action/       # Pending-action store (human-in-the-loop approvals)
  audit/                # Audit log (signing requests, session events)
  db/                   # Drizzle ORM schema and migrations
  util/, utils/         # Shared helpers
  test/
    helpers/            # Test infrastructure (SSH server, app, MCP/WS clients)
    integration/        # Integration tests by category
client/                 # SvelteKit frontend app (adapter-static)
  src/
    app.html            # HTML shell
    app.css             # Global styles (CSS variables, shared classes)
    service-worker.ts
    lib/
      stores/           # Svelte stores (ws, endpoints, keys, webauthn, auth)
      components/       # Reusable components (Terminal, Sidebar)
      utils/            # Utilities (FIDO signing)
    routes/
      +layout.svelte    # Root layout (sidebar + mobile nav)
      +page.svelte      # Terminal view (default route)
      admin/            # Admin views
      audit/            # Audit log view
      auth/callback/    # OAuth redirect target — exchanges code for tokens
      observer/         # Multi-session grid view
      passkey-invite/   # Passkey invite flow
      register/         # Initial admin registration
      session/          # Session detail

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rado0x54/ShellWatch](https://github.com/rado0x54/ShellWatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

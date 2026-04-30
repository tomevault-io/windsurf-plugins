---
trigger: always_on
description: PizzaPi is a self-hosted web interface and relay server for the [`pi` coding agent](https://github.com/badlogic/pi-mono). It streams live agent sessions to any browser and allows remote interaction from mobile or desktop without needing terminal access.
---

# PizzaPi — Agent Guide

PizzaPi is a self-hosted web interface and relay server for the [`pi` coding agent](https://github.com/badlogic/pi-mono). It streams live agent sessions to any browser and allows remote interaction from mobile or desktop without needing terminal access.

---

## Repository Layout

```
packages/
  cli/      CLI wrapper — launches pi with PizzaPi extensions and the runner daemon
  server/   Bun HTTP + WebSocket relay server (auth, session relay, attachments)
  ui/       React 19 PWA web interface (Vite, TailwindCSS v4, Radix UI / shadcn)
  tools/    Shared agent tools (bash, read-file, write-file, search, toolkit)
  docs/     Starlight (Astro) documentation site — https://pizzaface.github.io/PizzaPi/
  npm/      npm distribution — builds & publishes `npx pizzapi` packages

docker/     Docker Compose (redis + server services)
patches/    Bun patches for upstream pi packages (auto-applied on bun install)
```

Build order: `tools` → `server` → `ui` → `cli`.

---

## Documentation Site

User-facing documentation lives in `packages/docs/` — a [Starlight](https://starlight.astro.build/) (Astro) site deployed to GitHub Pages.

- **Source**: `packages/docs/src/content/docs/` (`.mdx` files)
- **Config**: `packages/docs/astro.config.mjs`
- **Build**: `cd packages/docs && bun run build`
- **Dev**: `cd packages/docs && bun run dev`
- **Live site**: https://pizzaface.github.io/PizzaPi/

**When changing CLI commands, flags, config options, or self-hosting behavior, always update the corresponding docs pages:**

| Topic | Doc page |
|-------|----------|
| CLI commands & flags | `running/cli-reference.mdx` |
| Config, env vars | `customization/configuration.mdx` |
| MCP servers | `customization/mcp-servers.mdx` |
| Hooks | `customization/hooks.mdx` |
| Skills | `customization/skills.mdx` |
| Agent definitions | `customization/agent-definitions.mdx` |
| Claude plugins | `customization/claude-plugins.mdx` |
| Subagents | `customization/subagents.mdx` |
| `pizza web`, Docker | `deployment/self-hosting.mdx` |
| Tailscale HTTPS | `deployment/tailscale.mdx` |
| Runner daemon | `running/runner-daemon.mdx` |
| Installation | `start-here/installation.mdx` |
| Sandbox & safe mode | `security/sandbox.mdx` |
| Architecture | `reference/architecture.mdx` |
| Server env vars | `reference/environment-variables.mdx` |

The README is intentionally slim — it has a quick start and links to the docs site. **Do not duplicate detailed docs in the README.**

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Runtime / package manager | **Bun** (required — not Node/npm/yarn) |
| Language | TypeScript (strict mode, ESM throughout) |
| Server | Bun.serve, better-auth, Kysely + SQLite, Redis, web-push |
| UI | React 19, Vite 6, TailwindCSS v4, Radix UI, shadcn/ui, streamdown |
| Agent core | `@mariozechner/pi-coding-agent`, `@mariozechner/pi-ai`, `@mariozechner/pi-tui` |

---

## Common Commands

```bash
# Install dependencies
bun install

# Build everything
bun run build

# Development (server + UI, hot-reload)
bun run dev

# Type-check all packages
bun run typecheck

# Run DB migrations
bun run migrate

# Clean all dist/ directories
bun run clean
```

---

## Development Notes

- **Always use `bun`** — no Node, npm, yarn, or pnpm.
- **Build order**: `tools` must be built before `server` or `cli`; `ui` can be built in parallel with `server`.
- **TypeScript**: run `bun run typecheck` to check all packages at once.
- **Patches**: Never edit files inside `node_modules` directly — changes go in `patches/` and are applied via `bun install`.
- **Redis** is required for the server. For local dev without Docker: `redis-server` or `docker compose up redis`.
- **Do not repoint sandbox/test harnesses at an existing user or production Redis instance** (for example `redis://127.0.0.1:6379`) without explicit user permission. Sandboxes must use their own isolated Redis and must not assume the user's local Redis is safe to reuse.
- **Database migrations**: run `bun run migrate` after schema changes. DB file is `packages/server/auth.db`.
- **UI + TUI for every feature**: Custom features should include both a **web UI** component (in `packages/ui`) and **TUI/CLI** support (agent tools in `packages/cli`). Backend-only features without a UI are incomplete — users interact through the web interface, not just agent tools. When adding a new capability, ask: "Can the user configure/see/use this from the web UI?" If not, add it.

---

## Upstream Patches

PizzaPi patches two upstream pi packages via `patchedDependencies` in the root `package.json`. Patches live in `patches/` and are auto-applied on `bun install`. See `patches/README.md` for full details.

### @mariozechner/pi-coding-agent

- **Session control:** Exposes `newSession()` / `switchSession()` on the extension API so the remote extension can trigger `/new` and `/resume` from the web UI.
- **Version check removal:** Disables the npm registry version check and "Update Available" notification (irrelevant for PizzaPi's headless runner).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pizzaface/PizzaPi](https://github.com/Pizzaface/PizzaPi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

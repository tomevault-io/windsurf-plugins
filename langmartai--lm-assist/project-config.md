---
trigger: always_on
description: Provides 3 tools via stdio transport (server name: `lm-assist`):
---

# lm-assist

Monorepo for the LM Assistant — a web UI for managing Claude Code sessions, with a backend API for session management, knowledge, and hub connectivity.

## Structure

```
lm-assist/
├── core/                    ← Backend API (TypeScript, dev :3200 / prod :3100)
│   ├── src/
│   │   ├── api/             ← API helper implementations (sessions, agent, tasks)
│   │   ├── checkpoint/      ← Git checkpoint management
│   │   ├── hub-client/      ← Hub WebSocket client (relay, sync)
│   │   ├── knowledge/       ← Knowledge generation pipeline
│   │   ├── mcp-server/      ← MCP server + tools (search, detail, feedback)
│   │   ├── routes/core/     ← Route files and endpoints
│   │   ├── search/          ← BM25 + text scoring
│   │   ├── types/           ← Shared TypeScript types
│   │   ├── utils/           ← Git, JSONL, path utilities
│   │   └── vector/          ← Embeddings + Vectra vector store
│   ├── hooks/               ← Hook scripts (statusline, context-inject)
│   ├── scripts/             ← tmux-autostart.sh
│   ├── package.json
│   └── tsconfig.json
├── web/                     ← Web UI (Next.js 16, dev :3948 / prod :3848)
│   ├── src/
│   │   ├── app/             ← Next.js App Router pages
│   │   ├── components/      ← React components
│   │   ├── contexts/        ← React contexts
│   │   ├── hooks/           ← Custom React hooks
│   │   ├── lib/             ← API clients, utilities
│   │   └── stores/          ← Zustand stores
│   ├── package.json
│   └── next.config.ts
├── core.sh                  ← Service manager (start/stop/restart/status)
├── package.json             ← Workspace root
├── .env.example
└── CLAUDE.md
```

## Commands

```bash
./core.sh              # Interactive menu
./core.sh start        # Start API + Web (auto-builds if needed)
./core.sh stop         # Stop all services
./core.sh restart      # Restart all services
./core.sh status       # Show service status + health check
./core.sh build        # Compile TypeScript (core)
./core.sh clean        # Clean and rebuild
./core.sh test         # Test API endpoints
./core.sh hub start    # Connect Hub Client
./core.sh hub stop     # Disconnect Hub Client
./core.sh hub status   # Hub connection info
./core.sh logs [core|web]  # View logs
```

**IMPORTANT: Always use `./core.sh` to manage services. Do not use direct npm/node commands.**

After modifying TypeScript in `core/src/`, rebuild with `./core.sh build` (or `./core.sh restart` which auto-builds if outdated).

## Dev/Prod Port Separation

Dev (repo) and prod (npm package) use **separate port spaces** so both can run simultaneously:

| Mode | Core API | Web UI | Managed by |
|------|----------|--------|------------|
| **Dev** | 3200 | 3948 | `./core.sh start/stop` (this repo) |
| **Prod** | 3100 | 3848 | `lm-assist start/stop` (npm package) |

**Use `./core.sh` for development** — build, start, test, and iterate on this repo. Use `lm-assist` CLI for managing the prod npm-installed version. Never use `lm-assist` to manage dev services or `./core.sh` to manage prod.

`./core.sh status` shows both environments side-by-side.

**Port detection methods by component:**
- `core.sh` — hardcoded dev defaults (3200/3948)
- TypeScript (cli.ts, service-manager, rest-server, hub-client, etc.) — `__dirname.includes('node_modules')` → prod (3100), else dev (3200)
- Hook + MCP + Statusline — reads `devModeEnabled` from `~/.claude-code-config.json`; when `devModeEnabled=true`, these components talk to the dev API (:3200) instead of prod (:3100)
- Web UI SSR — `NEXT_PUBLIC_LOCAL_API_PORT` env var (set by core.sh at build + start time)
- Web UI client — `NEXT_PUBLIC_LOCAL_API_PORT` baked in at `next build` time, plus `window.location.port` for self-referencing URLs

**When adding new port references:** never hardcode `3100` or `3848`. Use the appropriate detection method for the component type. For core TypeScript, use the `__dirname.includes('node_modules')` pattern.

### Testing After Code Changes

After modifying and rebuilding (`./core.sh build`), restart **dev** services:
```bash
./core.sh restart          # Restarts on dev ports 3200/3948
./core.sh status           # Verify both dev and prod status
```

Test the dev API: `curl http://localhost:3200/health`
Test the dev web: open `http://localhost:3948`

**Prod stays untouched** — `./core.sh restart` only affects dev ports. To test prod, use `lm-assist restart`.

### Browser Testing (Remote / MCP)

The browser automation MCP (Claude in Chrome) may run on a **different machine** than the dev server. When testing the web UI via browser:

1. Get this machine's IP: `hostname -I | awk '{print $1}'`
2. Use the IP (not `localhost`) in browser URLs: `http://<IP>:3948`
3. The core API also binds to `0.0.0.0`, so `http://<IP>:3200/health` works for remote testing
4. When navigating in browser automation tools, always use the IP-based URL for cross-machine access

## Architecture

### Core API (`core/`)

The backend is a raw Node.js HTTP server (no Express/Hono runtime — Hono is a dependency but the server uses `http.createServer` directly). Routes are modular: each `*.routes.ts` file exports an array of `{ method, pattern, handler }` objects matched via regex.

**Key components:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langmartai/lm-assist](https://github.com/langmartai/lm-assist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->

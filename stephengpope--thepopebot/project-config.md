---
trigger: always_on
description: Technical reference for AI assistants modifying the thepopebot NPM package source code.
---

# thepopebot — Package Source Reference

Technical reference for AI assistants modifying the thepopebot NPM package source code.

**Architecture**: Event Handler (Next.js) creates `agent-job/*` branches → launches Docker agent container locally (Claude Code, Pi, etc.) → task executed → PR created → auto-merge → notification. Agent jobs log to `logs/{JOB_ID}/`.

## Deployment Model

The npm package (`api/`, `lib/`, `config/`, `bin/`) is published to npm. In production:

- **Event handler**: Docker image bakes the npm package, Next.js app source (`web/`), and `.next` build output. User project directories (`agent-job/`, `event-handler/`, `skills/`, `.env`, `data/`, etc.) are individually volume-mounted into `/app`. The full project is also mounted at `/project` for git access. Runs `server.js` via PM2 behind Traefik reverse proxy.
- **`lib/paths.js`**: Exports `PROJECT_ROOT` (`process.cwd()`). This is how the installed npm package finds the volume-mounted user project files.
- **Agent-job containers**: Ephemeral Docker containers clone `agent-job/*` branches separately — use named volumes for workspace. See `docker/CLAUDE.md`.
- **Local install**: Gives users CLI tools (`init`, `setup`, `upgrade`) and configuration scaffolding.

## Package vs. Templates — Where Code Goes

All event handler logic, API routes, library code, and core functionality lives in the **npm package** (`api/`, `lib/`, `config/`, `bin/`). This is what users import when they `import ... from 'thepopebot/...'`.

The `templates/` directory contains **only files that get scaffolded into user projects** via `npx thepopebot init`. Templates are for user-editable configuration and thin wiring — things users are expected to customize or override. Never add core logic to templates.

**When adding or modifying event handler code, always put it in the package itself (e.g., `api/`, `lib/`), not in `templates/`.** Templates should only contain:
- Configuration files users edit (`agent-job/SOUL.md`, `agent-job/CRONS.json`, `event-handler/TRIGGERS.json`, etc.)
- GitHub Actions workflows
- Docker compose (`docker-compose.yml`)
- CLAUDE.md files for AI assistant context in user projects

Next.js app source files (`app/`, `next.config.mjs`, `server.js`, etc.) live in `web/` at the package root. These are built into the Docker image — NOT scaffolded to user projects.

### Managed Paths

Files in managed directories are auto-synced (created, updated, **and deleted**) by `init` to match the package templates exactly. Users should not edit these files — changes will be overwritten on upgrade. Managed paths are defined in `bin/managed-paths.js`:

- `.github/workflows/` — CI/CD workflows
- `docker-compose.yml`, `.dockerignore` — Docker config
- `.gitignore` — Git ignore rules
- `CLAUDE.md` — AI assistant context

## Directory Structure

```
/
├── api/                        # GET/POST handlers for all /api/* routes
├── lib/
│   ├── actions.js              # Shared action executor (agent, command, webhook)
│   ├── cron.js                 # Cron scheduler (loads CRONS.json)
│   ├── triggers.js             # Webhook trigger middleware (loads TRIGGERS.json)
│   ├── paths.js                # Exports PROJECT_ROOT (process.cwd())
│   ├── ai/                     # LLM integration (agent, model, tools, streaming)
│   ├── auth/                   # NextAuth config, helpers, middleware, server actions, components
│   ├── channels/               # Channel adapters (base class, Telegram, factory)
│   ├── chat/                   # Chat route handler, server actions, React UI components
│   ├── cluster/                # Worker clusters (roles, triggers, Docker containers)
│   ├── code/                   # Code workspaces (server actions, terminal view, WebSocket proxy)
│   ├── containers/             # Container SSE streaming (Docker container status)
│   ├── db/                     # SQLite via Drizzle (schema, migrations, api-keys)
│   ├── tools/                  # Job creation, GitHub API, Telegram, Docker, Whisper
│   ├── voice/                  # Voice input (AssemblyAI streaming transcription)
│   └── utils/
│       └── render-md.js        # Markdown {{include}} processor
├── config/
│   ├── index.js                # withThepopebot() Next.js config wrapper
│   └── instrumentation.js      # Server startup hook (loads .env, starts crons)
├── bin/                        # CLI entry point (init, setup, reset, diff, upgrade)
├── setup/                      # Interactive setup wizard
├── web/                        # Next.js app source (baked into Docker image, NOT scaffolded)
│   ├── app/                    # Next.js app directory (pages, layouts, routes)
│   ├── server.js               # Custom Next.js server with WebSocket proxy
│   ├── next.config.mjs         # Next.js config wrapper
│   ├── instrumentation.js      # Server startup hook
│   ├── middleware.js            # Auth middleware
│   └── postcss.config.mjs      # PostCSS/Tailwind config
├── templates/                  # Scaffolded to user projects (see rule above)
├── docs/                       # Extended documentation
└── package.json
```

## NPM Package Exports


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stephengpope/thepopebot](https://github.com/stephengpope/thepopebot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

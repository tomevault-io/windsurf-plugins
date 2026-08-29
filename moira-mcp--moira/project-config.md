---
trigger: always_on
description: Agent guide for the Moira repository. Read this before making changes. Claude Code
---

# AGENTS.md

Agent guide for the Moira repository. Read this before making changes. Claude Code
reads `CLAUDE.md`, which imports this file with `@AGENTS.md`.

Before beginning any development work, read `CONTRIBUTING.md` completely. It is the
source of truth for coordinating work through GitHub issues, claiming or releasing
an issue, pull-request linkage, and the contributor/maintainer handoff. Do not start
implementation from an issue until that contract says the issue is available and
the claim has been confirmed.

## What Moira Is

Moira is a node-graph **Agent Workflow Engine**. It guides AI agents (Claude, GPT,
custom agents) through multi-step processes via the **MCP protocol**, giving each
step a clear directive (what to do) and a completion condition (when it's done),
validated before the agent may proceed. The primary users are AI agents; the Web UI
is a supplementary tool for managing workflows.

See `docs/VISION.md` for the full product vision.

## Repository Layout

npm-workspace monorepo:

```
packages/
├── workflow-engine/   # Core node-graph execution engine
├── mcp-server/        # MCP protocol HTTP server (the MCP tools)
├── web-backend/       # Express API for workflow management
├── web-frontend/      # React UI for workflow visualization
├── shared/            # Database (Drizzle), Better Auth, logging, config
└── docs/              # Astro 5 + Starlight documentation site (EN + RU)
workflows/             # Bundled workflow catalog (workflows/production/public/)
config/                # Dockerfile, nginx, supervisord, prompts
scripts/               # DB init, migrations, secret bootstrap, health checks
tests/                 # unit / workflow / integration / api / e2e / mcp-tools
docs/                  # Internal developer documentation
```

For where each topic is documented, see the **Documentation Map** in `README.md`.

## Build & Run (fresh clone)

The app runs as a single Docker container built from a **baked image** — there is no
live-source mode. Code changes (engine, MCP server, backend, frontend, bundled
workflow catalog) reach a running instance only by rebuilding the image and
recreating the container; `docker restart` keeps the old build. Check which commit an
image was built from with `docker exec <container> cat /app/BUILD_INFO`. Ports are
env-driven (`MOIRA_PORT`, default 8080).

The local instance runs one of two images — registry or locally built:

```bash
cp .env.example .env          # defaults work locally; review host, port, and artifact domain for another host

# A. published image from the registry (docker-compose.yml default)
docker compose up -d          # pulls ghcr.io/moira-mcp/moira:latest

# B. image built from this checkout — in docker-compose.yml comment out the
#    `image:` line and uncomment the `build:` block, then:
docker compose up -d --build  # builds config/Dockerfile

# Web UI: http://localhost:8080   Docs: /docs   MCP: /mcp
```

Both mount the host `./data` volume, so the SQLite DB (users, settings, uploaded
workflows) and execution storage survive a rebuild. Option B is the only way to run
uncommitted source changes on the local instance.

`npm run docker:restart` is a **contributor dev helper, not how the local instance
runs**. It reads `.env.local` (copy `.env.local.example`) and builds a _separate_
image/container/port (`DOCKER_IMAGE_NAME`, `DOCKER_CONTAINER_NAME`, `DOCKER_PORT`),
bind-mounts `./workflows` into the container and leaves `/app/data` ephemeral inside
the container — that DB is discarded on every rebuild. Never assume `docker:restart`
updates the instance an MCP client is pointed at.

```bash
npm run docker:restart        # build + start the dev container
npm run docker:stop           # stop it
```

Self-host users do not need `docker:restart`; `docker compose up -d` is enough.

## Testing

Run tests ONLY through the npm scripts (they set the correct env, DB paths, and
output files). NEVER call `jest` / `npx jest` / `playwright test` directly.

```bash
npm test                 # all suites
npm run test:unit        # unit (in-memory DB)
npm run test:workflow    # workflow scenarios (test DB)
npm run test:integration # integration (test DB)
npm run test:api         # API (HTTP → local Docker)
npm run test:mcp-tools   # MCP tools (HTTP → local Docker)
npm run test:e2e         # E2E (Playwright → local Docker)
```

Test databases:

- Unit → in-memory.
- Integration / workflow → `./data/test-integration.db`.
- API / E2E / MCP → `./data/moira.db` (the Docker container DB).

If a script does not support what you need, say so and propose extending the
script — do not work around it with direct `npx` calls.

### Test Quality Rules

Tests are the only regression guard between sessions. Every test must verify
concrete functionality and fail when it breaks. Forbidden antipatterns:

1. **No-op assertions** — `expect(true).toBe(true)`.
2. **Conditional assertions** — `if (visible) { expect(...) }` (assertion may never run).
3. **Empty stub tests** — a `test()` with no assertions / only a TODO.
4. **Inline algorithm copy** — re-implementing production logic in the test instead of asserting concrete cases.
5. **Performance test without a threshold** — measuring time but not asserting it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moira-mcp/moira](https://github.com/moira-mcp/moira) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->

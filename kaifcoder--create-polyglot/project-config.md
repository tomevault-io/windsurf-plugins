---
trigger: always_on
description: Purpose: CLI tool (`bin/index.js`) that scaffolds polyglot monorepos with Node.js, Python/FastAPI, Go, Spring Boot Java, Next.js, Remix, Astro, and SvelteKit. Supports Turborepo/Nx presets, Docker orchestration, hot reload, plugin system, and admin dashboard with real-time monitoring.
---

# Copilot Instructions for create-polyglot

Purpose: CLI tool (`bin/index.js`) that scaffolds polyglot monorepos with Node.js, Python/FastAPI, Go, Spring Boot Java, Next.js, Remix, Astro, and SvelteKit. Supports Turborepo/Nx presets, Docker orchestration, hot reload, plugin system, and admin dashboard with real-time monitoring.

## Core Architecture
- **Entrypoint**: `bin/index.js` (ESM) routes subcommands to modular handlers in `bin/lib/`:
  - `scaffold.js` - Project initialization, service/plugin/library add/remove
  - `dev.js` - Local development runner with health checks
  - `hotreload.js` - Language-specific hot reload orchestration (nodemon, uvicorn, go run, spring-boot)
  - `admin.js` - HTTP dashboard server with WebSocket log streaming (chokidar-based file watching)
  - `logs.js` - Log management (viewing, filtering, cleanup, file watching with `LogFileWatcher`)
  - `service-manager.js` - Process lifecycle management (start/stop/status services)
  - `plugin-system.js` - Hookable-based plugin lifecycle (60+ hook points)
  - `resources.js` - System resource monitoring (pidusage, systeminformation)
  - `ui.js` - CLI rendering helpers (tables via chalk)

- **Configuration**: `polyglot.json` manifest drives all commands (services, ports, preset, packageManager, plugins, sharedLibs)
- **Templates**: `templates/<type>` copied verbatim; Spring Boot special handling (`application.properties.txt` → `.properties`)
- **Service Model**: Array of `{ type, name, port, path }` where path is `services/<name>` (new) or `apps/<name>` (legacy)
- **Default Ports**: frontend 3000, node 3001, go 3002, java 3003, python 3004, remix 3005, astro 3006, sveltekit 3007
- **Reserved Names**: `scripts`, `packages`, `apps`, `node_modules`, `docker`, `compose`, `compose.yaml` rejected during validation
- **Port Uniqueness**: Enforced at init and add; conflicts abort early with clear error

## Key Workflows

### Scaffolding (`init` command)
1. Parse CLI args (commander) → gather missing via `prompts` (skip if `--yes`)
2. Call plugin hook `before:init`
3. Validate services: check reserved names, port conflicts, valid types
4. Create directory skeleton: `<project>/services/*`, `packages/shared`, `.polyglot/plugins/`
5. Write root artifacts: `package.json`, `polyglot.json`, `.eslintrc.cjs`, `.prettierrc`, README
6. Copy templates from `templates/<type>` → `services/<name>` (conditional `create-next-app` for frontend if `--frontend-generator`)
7. Generate Dockerfiles (inline logic, no external lib) + `compose.yaml` with `app-net` network
8. Optional: generate `.github/workflows/ci.yml` if `--with-actions`
9. Optional: `git init` (non-fatal on failure)
10. Install deps via `execa` unless `--no-install` (warns on failure, doesn't abort)
11. Call plugin hook `after:init`
12. Initialize service logs (`.logs/<date>.log`)

### Service Management
- **Add service**: Validate name/port → update `polyglot.json` → copy template → generate Dockerfile → call plugin hooks
- **Remove service**: Prompt confirmation (unless `--yes`) → delete files (unless `--keep-files`) → update `polyglot.json` → rebuild Docker compose
- **List services**: Read `polyglot.json` → render table or JSON

### Development
- **`dev` command**: Spawns processes per service type (npm run dev for node/frontend, uvicorn for python, go run for go, mvn spring-boot:run for java) + admin dashboard on port 9000
- **`dev --docker`**: Delegates to `docker compose up --build`
- **`hot` command**: Language-specific file watchers with debounced restarts (400ms)
- **`admin` command**: Starts HTTP server with resource monitoring + WebSocket log streaming (chokidar watches `.logs/*.log`)

### Plugin System
- Hooks execute via `hookable` library (before/after: init, service:add/remove, dev:start/stop, docker:build, hotreload:*, admin:*, logs:*)
- Plugins discovered from `.polyglot/plugins/` (local) or `node_modules` (npm)
- Dependency resolution via `dependencies` array in plugin metadata
- Enable/disable via `polyglot.json` plugins config

## Project Conventions
- **Module system**: ESM (`"type": "module"` in root package.json). All imports use `.js` extensions
- **Test runner**: Vitest (`npm test` → `vitest run && npm run test:cleanup`). Tests in `tests/*.test.js` with 30s+ timeouts for CLI ops
- **Output style**: All user-facing messages use `chalk` with consistent emoji prefixes:
  - Info: `chalk.cyan('🚀 ...')` or `chalk.yellow('⚠️  ...')`
  - Success: `chalk.green('✅ ...')`
  - Errors: `chalk.red('❌ ...')` followed by `process.exit(1)` for hard failures
  - Service logs: color per service via hash of name (see `colorFor()` helper in dev.js, hotreload.js)
- **Interactive defaults** (when `--yes`): projectName='app', services=['node'], preset=none, packageManager=npm, git=false
- **File operations**: Use `fs-extra` for async file ops (mkdirp, copy, readJson, writeJson). Use `execa` for spawning commands (not raw child_process except in service-manager.js)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaifcoder/create-polyglot](https://github.com/kaifcoder/create-polyglot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

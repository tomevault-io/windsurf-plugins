---
trigger: always_on
description: This project uses a **unified single container** for all environments (production, development, and CI). The container runs the Python API, Next.js web UI, and nginx reverse proxy together. All traffic goes through port 4420 on the host, mapped to nginx on port 3000 inside the container.
---

# FiestaBoard — Claude Code Context

## Single-Container Architecture

This project uses a **unified single container** for all environments (production, development, and CI). The container runs the Python API, Next.js web UI, and nginx reverse proxy together. All traffic goes through port 4420 on the host, mapped to nginx on port 3000 inside the container.

### Architecture Overview

- **Single Dockerfile** (`Dockerfile`): Multi-stage build producing one image with API + UI + nginx
- **Production** (`docker-compose.yml`): Single container, no volume mounts
- **Development** (`docker-compose.dev.yml`): Same container with volume mounts for Python hot-reload, uses `start-dev.sh` (uvicorn `--reload`)
- **CI** (`.github/workflows/ci.yml`): Tests run directly on the GitHub Actions host (not in Docker) for speed; Docker image build is verified separately

### Core Rules

1. **NEVER run the API server (`src/api_server.py` or `python -m src.api_server`) directly on the host machine**
   - Always use Docker containers via `docker-compose` commands

2. **NEVER run the web UI (`npm run dev` in `web/` directory) directly on the host machine**
   - Always use Docker containers via `docker-compose` commands

3. **NEVER install Python dependencies locally or suggest running `pip install`**
   - All Python dependencies are managed within Docker containers

4. **NEVER install Node.js dependencies locally or suggest running `npm install` in the web/ directory**
   - All Node.js dependencies are managed within Docker containers

### Development Workflow

- **Starting the dev container**: Use `/start` command or `docker-compose -f docker-compose.dev.yml up`
- **Stopping**: Use `/stop` command or `docker-compose -f docker-compose.dev.yml down`
- **Restarting**: Use `/restart` command (stops, rebuilds with --no-cache, restarts)
- **Building**: Use `/build` command to rebuild images without restarting
- **Running tests**: Use `/test-api` or `/test-web` commands to run tests inside the Docker container
- **Debugging**: Use `docker-compose logs -f` or `docker-compose exec` for interactive debugging
- **Checking status**: Use `docker-compose ps` to see container status
- **Code changes**: Edit code on host machine; Python changes auto-reload via volume mounts. UI changes require a container rebuild.
- **Installing dependencies**:
  - For Python: Update `requirements.txt` and rebuild the container
  - For Node.js: Update `web/package.json` and rebuild the container

### Testing Guidelines

- **Local development**: Tests run inside the Docker container
  - Use `docker-compose -f docker-compose.dev.yml exec fiestaboard pytest` for Python tests
  - Use `docker-compose -f docker-compose.dev.yml run --rm --profile test web sh -c "npm ci && npm test"` for web tests
- **CI**: Tests run directly on the GitHub Actions host for speed (Python and Node.js installed natively)
- Re-testing after changes should also be done in the container

### Exceptions

The only code that may run locally:
- Utility scripts explicitly marked for local execution

### Available Commands

- `/setup` - Check and install prerequisites (Homebrew, Docker) and configure environment
- `/start` - Start the dev container
- `/stop` - Stop the dev container
- `/restart` - Stop, rebuild with --no-cache, and restart the container
- `/redeploy` - Full rebuild with --no-cache and restart (alias for restart)
- `/redeploy-quick` - Quick rebuild (with cache) and restart
- `/build` - Rebuild Docker image without restarting
- `/status` - Show status of the Docker container
- `/logs` - View Docker container logs
- `/test-api` - Run API tests in Docker
- `/test-web` - Run web tests in Docker

### When Suggesting Commands

Always suggest Docker-based commands:
- ✅ Use `/start` and `/stop` commands to control the container
- ✅ Use `/restart` command for full redeployment with clean rebuild
- ✅ Use `/build` command to rebuild the image
- ✅ Use `/test-api` or `/test-web` commands for testing
- ✅ `docker-compose -f docker-compose.dev.yml up`
- ✅ `docker-compose -f docker-compose.dev.yml exec fiestaboard pytest`
- ✅ `docker-compose -f docker-compose.dev.yml run --rm --profile test web sh -c "npm ci && npm test"`
- ❌ `python src/api_server.py`
- ❌ `cd web && npm run dev`
- ❌ `pip install -r requirements.txt`

## Plugin Architecture

This project uses a **plugin-based architecture** for data source integrations. Each plugin is self-contained with its own code, configuration, tests, and documentation.

### Plugin vs Platform Code

- **Plugins** (`plugins/`): Self-contained data source integrations (weather, transit, stocks, etc.)
- **Platform** (`src/`): Core FiestaBoard infrastructure (API server, template engine, board client)

### Key Plugin Concepts

- Plugins live in `plugins/<plugin_id>/` directories
- Each plugin has a `manifest.json` defining metadata, settings schema, and variables
- Plugins implement the `PluginBase` class from `src/plugins/base.py`
- Plugin IDs must be unique and match the directory name
- All plugins require tests with >80% coverage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fiestaboard/FiestaBoard](https://github.com/Fiestaboard/FiestaBoard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

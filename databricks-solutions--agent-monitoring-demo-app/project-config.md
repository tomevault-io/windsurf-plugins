---
trigger: always_on
description: - Use `uv` for Python package management instead of `pip` directly
---

# Project Memory

## Package Management
- Use `uv` for Python package management instead of `pip` directly
- This project uses uv for dependency management and virtual environment handling
- When changing Python dependencies, always use `uv add` or `uv remove` commands instead of editing pyproject.toml directly

## Frontend Setup
- **Always use Bun for frontend operations** - faster than npm/yarn and eliminates dependency conflicts
- Client uses shadcn/ui components with proper TypeScript configuration
- Development: `bun start` or `bun dev` in the client directory
- Build process: `bun run build` in the client directory  
- Package management: Use `bun add` and `bun remove` instead of npm
- shadcn components can be added with: `npx shadcn@latest add <component-name>`
- **TypeScript path aliases**: Use canonical `@/` imports (e.g., `import { Button } from "@/components/ui/button"`)
- **Build system**: Vite and TypeScript are configured for "@/" alias support automatically

## Development Server Management
- Use "start server" command to run the development server using `./watch.sh` in a detached screen session
- The dev server runs both the FastAPI backend (port 8000) and React frontend (Vite dev server) with hot reload
- Server status can be checked with "server status" or "is server running" commands
- Server can be stopped with "stop server" or "kill server" commands
- Screen session is named "lha-dev" and can be accessed directly with `screen -r lha-dev`
- When server is running, you can test changes immediately without manual restarts
- The server automatically opens http://localhost:8000 when started

## API Endpoint Testing Methodology
When adding or testing endpoints, use this workflow:
1. **Test the endpoint** with curl:
   ```bash
   curl -X POST http://localhost:8000/api/agent \
     -H "Content-Type: application/json" \
     -d '{"inputs": {"messages": [{"role": "user", "content": "What is Databricks?"}]}}'
   ```
2. **Check server logs** immediately after:
   ```bash
   screen -S lha-dev -X hardcopy /tmp/server_logs.txt && tail -30 /tmp/server_logs.txt
   ```
3. **Verify response** includes expected fields (response, trace_id for agent endpoint)
4. **Confirm MLflow tracing** is working (trace_id should be present)

## Hot Reload Development Workflow
The development server supports real-time code changes:
1. **Add debug prints** to server code (e.g., `print(f"🔥 ENDPOINT HIT: {data}")`)
2. **uvicorn auto-reloads** the server when Python files change
3. **Test immediately** with curl - no manual restart needed
4. **Verify functionality** through response content and status codes
5. **Clean up debug code** when done testing
Note: Screen capture may not always show real-time output clearly, but process monitoring and endpoint testing provide reliable verification.

## Development Server Troubleshooting
- **Profile errors in watch.sh**: The script handles optional DATABRICKS_CONFIG_PROFILE - if not set, uses default auth
- **Screen output capture**: Use `ps aux | grep uvicorn` and direct endpoint testing rather than relying solely on screen hardcopy
- **Port conflicts**: Check `lsof -i :8000` to verify server is listening correctly
- **Process verification**: Multiple uvicorn processes are normal (parent/child from --reload mode)

## Testing
- When making changes to the agent code in `server/agents/databricks_assistant.py`, use `./test_agent.sh` (or `uv run python test_agent.py`) to test the agent directly without starting the full web application
- This executes the actual databricks_assistant.py code and allows for faster iteration and debugging of agent behavior
- The test script shows both the full JSON response and just the content for easier reading
- For full UI testing, use the development server (see Development Server Management section)

## Setup
- Use `./setup.sh` to interactively create/configure the .env.local file with all required environment variables

## Code Formatting
- Use `./fix.sh` to format all code according to the project's style guidelines
- This runs ruff formatting/linting for Python files and prettier for TypeScript/JavaScript files
- Run this before committing code to ensure consistent formatting

## Build System & Generated Files
- **Never commit build artifacts**: `client/build/` contains Vite build output (ignored in .gitignore)
- **API client is auto-generated**: `client/src/fastapi_client/` is generated from OpenAPI spec via `uv run python -m scripts.make_fastapi_client`
- **Lock files**: `uv.lock` should be committed to ensure reproducible dependency versions across all developers
- **Development workflow**: The `./watch.sh` script automatically regenerates the API client when backend changes
- When adding new FastAPI endpoints, the TypeScript client updates automatically

## Git Operations
- Use `git pp` instead of `git push` for pushing changes

## Deployment
- When the user says "deploy", use `./deploy.sh` command
- **Automated app.yaml configuration**: Deploy script automatically updates `app.yaml` with `MLFLOW_EXPERIMENT_ID` from `.env.local`
- **Automated verification**: After running deploy.sh, programmatically check deployment success by:
  1. Running `databricks apps list` to verify app appears

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [databricks-solutions/agent-monitoring-demo-app](https://github.com/databricks-solutions/agent-monitoring-demo-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->

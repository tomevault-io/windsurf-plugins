---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Malla - Meshtastic Mesh Health Web UI

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

Bootstrap, build, and test the repository:
- Install uv package manager: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Add uv to PATH: `export PATH="/home/runner/.local/bin:$PATH"`
- Install dependencies: `uv sync` -- takes 5 seconds on first run
- Install development dependencies: `uv sync --dev` -- takes <1 second if dependencies already exist

Test the codebase:
- Check test environment: `uv run python run_tests.py --check`
- Run unit tests: `uv run python run_tests.py unit -v` -- takes 1.4 seconds. NEVER CANCEL.
- Run integration tests: `uv run python run_tests.py integration -v` -- takes 4.3 seconds. NEVER CANCEL.
- Run full test suite: `make check` -- takes 3 minutes. NEVER CANCEL. Set timeout to 240+ seconds.
- Install Playwright browsers (for e2e tests): `uv run playwright install` -- takes 12 seconds

Lint and format code:
- Lint only: `make lint` -- takes 4.8 seconds
- Format code: `make format` -- takes <0.1 seconds  
- Check + test: `make check` -- runs lint + full test suite, takes 3 minutes. NEVER CANCEL.

Run the applications:
- ALWAYS run the bootstrapping steps first
- Web UI: `uv run malla-web` (development server, Flask)
- Web UI (production): `uv run malla-web-gunicorn` (Gunicorn WSGI server)
- MQTT capture: `uv run malla-capture` (requires MQTT broker configuration)
- Access web UI at: http://localhost:5008
- **Note**: Always use `uv run` commands. The wrapper scripts (`./malla-web`, `./malla-capture`) require the virtual environment to be activated.

## Validation

- **CRITICAL**: ALL TESTS MUST PASS - Do not break existing tests under any circumstances
- **MANDATORY**: ABSOLUTELY POSITIVELY RUN ALL OF THE TEST SUITE to verify your CHANGES ALL THE TIME
- **MANDATORY**: Always run `make check` before considering work finished - this runs the full CI pipeline (takes 3 minutes)
- **REQUIRED**: Run `make format` and `make lint` before committing or CI will fail
- **NON-NEGOTIABLE**: Fix all linting errors before committing - formatting issues can be auto-fixed with `make format`
- **NON-NEGOTIABLE**: YOU SHALL NOT BREAK THE TESTS - Test failures indicate broken functionality that must be debugged and fixed, not ignored
- **FAILURE**: Discovering test failures only when they run on CI is a failure worth -100 points
- **MANDATORY**: If there's something missing for you to run tests, dig until you've found them and update your own configuration if needed

**Complete validation workflow (MANDATORY before finishing):**
1. Format code: `make format` (auto-fixes formatting issues)
2. Lint code: `make lint` (must pass completely)
3. Run comprehensive CI checks: `make check` (ALL tests must pass - 430+ tests including 44 unit, 168 integration)
4. Manual validation: Start web UI `uv run malla-web` and verify basic functionality
5. Only commit changes after ALL validation passes

**Critical validation points:**
- Web UI starts and serves HTTP 200 on port 5008
- API endpoints return valid JSON responses
- Database gets created automatically
- **ALL unit tests pass (44 tests)**
- **ALL integration tests pass (168 tests)**
- E2E tests may fail in sandboxed environments due to missing browser dependencies (expected)
- ANY test regression indicates broken functionality that MUST be fixed

## Critical Timeouts and "NEVER CANCEL" Warnings

**NEVER CANCEL: Full test suite takes 3 minutes. Set timeout to 240+ seconds.**
**NEVER CANCEL: Build operations are fast (<10 seconds) but CI includes comprehensive testing.**
**NEVER CANCEL: Playwright browser installation takes 12 seconds.**

## Configuration

**IMPORTANT**: Copy configuration before running:
```bash
cp config.sample.yaml config.yaml
```

Configuration can be done via:
1. YAML file: `config.yaml` (recommended for development)
2. Environment variables: Prefix all settings with `MALLA_` (recommended for production)

Key settings:
- `mqtt_broker_address`: MQTT broker hostname/IP (required for capture tool)
- `database_file`: SQLite database location (default: "meshtastic_history.db")
- `host`/`port`: Web server bind address (default: 0.0.0.0:5008)

## Docker Deployment

For production deployment:
```bash
# Copy environment configuration
cp env.example .env
# Edit .env with your MQTT broker address and settings
docker-compose up -d
```

For development with local changes:
```bash
# Edit docker-compose.yml to uncomment 'build: .' lines
docker-compose up --build -d
```

## Common Tasks

### Repository Structure
```
├── src/malla/           # Main application code
│   ├── web_ui.py       # Flask web application entry point
│   ├── mqtt_capture.py # MQTT capture tool entry point
│   ├── routes/         # Web UI routes and API endpoints
│   ├── models/         # Data models and database interaction
│   ├── services/       # Business logic services  
│   ├── templates/      # Jinja2 HTML templates
│   └── static/         # CSS, JavaScript, and static assets
├── tests/              # Test suite (unit, integration, e2e)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zenitraM/malla](https://github.com/zenitraM/malla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CodeLeash is an opinionated full-stack development scaffold that demonstrates how to build web applications with Claude Code using strong guardrails, Test-Driven Development, and architectural enforcement. It includes a minimal "hello world" implementation that exercises all architectural patterns (repository, service, container DI, React root mounting with initial data).

## Technology Stack

- **Backend**: Python with FastAPI
- **Frontend**: React with TypeScript, Vite build system, Tailwind CSS
- **Database**: Supabase (PostgreSQL)
- **Authentication**: Supabase Auth with JWT tokens
- **Observability**: Prometheus metrics, OpenTelemetry tracing, Sentry error tracking

## Key Commands

### Running the Application

To run the application:

#### Development Mode

```bash
# Run both backend and frontend with hot reload
npm run dev
```

#### Preview Mode (Production Build)

```bash
# Build frontend and run the full application
npm run preview
```

#### Docker Mode (Production-like)

```bash
# Run in containers
docker compose up --build

# View logs
docker compose logs -f web
```

The application runs on http://localhost:8000 in all local modes.

**IMPORTANT FOR CLAUDE**: Always check if the server is running on port 8000 before testing.

### Important Restrictions

**IMPORTANT FOR CLAUDE**:

- NEVER deploy the application to production. Deployment is the user's responsibility.
- NEVER run Docker commands on production systems. Only provide guidance and troubleshooting advice.

### Key Development Commands

```bash
# Install dependencies
uv sync --all-extras        # All dependencies
uv add package-name         # Add new package

# Generate TypeScript types from Pydantic models
npm run types

# Database operations
supabase migration up       # Apply migrations
supabase db reset          # Reset database
```

### Initial Setup

```bash
# Install dependencies, configure environment, install git pre-commit hook
./init.sh
```

**IMPORTANT FOR CLAUDE**: When starting work in a fresh repo or worktree, run `./init.sh` first. It installs dependencies, configures the environment for the worktree, starts Supabase, and installs a git pre-commit hook that runs `npm run test:all` on every commit.

### Code Quality Checks

```bash
# Run all pre-commit hooks
npm run pre-commit
```

```bash
# Run pre-commit and all tests
npm run test:all
```

A git pre-commit hook (installed by `./init.sh`) runs `npm run test:all` automatically on every commit, which includes pre-commit checks, vitest, pytest, and e2e tests in parallel. `npm run pre-commit` remains available for running just the linting/formatting checks.

After making any code changes, prefer running pre-commit hooks to verify code quality, formatting, and type checking, over other tools. This ensures all changes meet the project's coding standards and prevents issues in the CI/CD pipeline.

### Running Tests

```bash
# Run all tests
npm run test:all           # Python + TypeScript + E2E
npm run test:python        # Python backend tests
npm test                   # TypeScript frontend tests
npm run test:e2e          # End-to-end tests in parallel (fully automated)
npm run test:e2e:serial   # End-to-end tests in serial mode (slower)

# Run specific test files
npm run test:python -- tests/unit/services/test_greeting_service.py
npm run test:python -- tests/unit/services/test_foo.py -k "test_name" -v
npm test -- src/components/GreetingList.test.tsx
npm run test:e2e -- tests/e2e/test_hello_world.py -k "test_name" -v
```

**IMPORTANT FOR CLAUDE**: Always run tests as above. Running tests via `uv run pytest` or `npx vitest` will fail with permissions errors.

**CRITICAL TEST OUTPUT REQUIREMENT FOR CLAUDE**:

**A PreToolUse hook BLOCKS any test command that contains `|`, `;`, or `>`.** Running `npm test ... | grep`, `npm run test:python ... | tail`, `npm run test:e2e > /tmp/output.txt`, or any piped/chained/redirected variant will be rejected by the hook and the command will not execute. Do not attempt workarounds -- just run the test command by itself with no pipes or redirection.

Test output in this project is already minimal and informative. Run test commands directly:

```bash
# Correct - runs directly, full output
npm run test:python -- tests/unit/services/test_foo.py -k "test_name" -v

# BLOCKED by hook - do not attempt
npm run test:python -- tests/unit/services/test_foo.py | tail -20
npm test -- src/components/Foo.test.tsx | grep FAIL
npm run test:e2e 2>&1 | head -50
npm run test:e2e > /tmp/test_output.txt 2>&1
```

**IMPORTANT FOR CLAUDE**:

- E2E tests are fully automated - never ask users to start the server manually
- E2E tests automatically build the frontend before running (in scripts/run_e2e_tests.py) - no need to run `npm run build` before e2e tests
- Unit tests enforce a 10ms timeout - expensive imports are prewarmed in conftest.py
- E2E test suites take up to 3 minutes to complete even when run in parallel

## Architecture Overview

### Core Models (in app/models/)

- **User**: User identity with email, full_name, timestamps

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cadamsdotcom/CodeLeash](https://github.com/cadamsdotcom/CodeLeash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

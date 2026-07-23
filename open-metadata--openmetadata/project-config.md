---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About OpenMetadata

OpenMetadata is a unified metadata platform for data discovery, data observability, and data governance. This is a multi-module project with Java backend services, React frontend, Python ingestion framework, and comprehensive Docker infrastructure.

For architecture deep dives, entity/repository/resource patterns, and end-to-end checklists for adding new entities or connectors, see [DEVELOPER.md](DEVELOPER.md).

## Architecture Overview

- **Backend**: Java 21 + Dropwizard REST API framework, multi-module Maven project
- **Frontend**: React + TypeScript, built with Webpack and Yarn; component library via `openmetadata-ui-core-components` (Tailwind CSS v4 with `tw:` prefix, react-aria-components foundation)
- **Ingestion**: Python 3.10-3.11 with Pydantic 2.x, 75+ data source connectors
- **Database**: MySQL (default) or PostgreSQL with Flyway migrations
- **Search**: Elasticsearch 7.17+ or OpenSearch 2.6+ for metadata discovery
- **Infrastructure**: Apache Airflow for workflow orchestration

## Environment Setup

### Python Virtual Environment (REQUIRED)

**You MUST activate the Python venv before any Python work.** OpenMetadata supports Python 3.10-3.11; 3.11 is recommended.

```bash
# First-time setup (creates venv at repo root):
# python3.11 -m venv env

# ALWAYS activate before running Python, make generate, make install_dev, etc:
source env/bin/activate

# Verify:
python --version   # Should show Python 3.10.x or 3.11.x
```

**In worktrees**: When Claude Code creates a Git worktree, the venv from the main repo is NOT copied. You need to either:
- Create a new venv in the worktree: `python3.11 -m venv env && source env/bin/activate && cd ingestion && make install_dev`
- Or symlink the main repo's venv: `ln -s /path/to/main-repo/env env`

### Initial Dev Environment Setup

After activating the venv, install all dependencies:

```bash
source env/bin/activate

# Install ingestion module with all dev dependencies (required before make generate)
cd ingestion
make install_dev_env            # Full dev environment (edit mode + all extras)
# OR for lighter install:
make install_dev                # Just dev dependencies
cd ..

# Generate Pydantic models from JSON schemas (required after schema changes)
make generate

# Install UI dependencies
make yarn_install_cache
```

### Other Environment Notes

- **Java**: Java 21 required. Use `mvn` (Maven) for backend builds.
- **Node/Yarn**: Use `yarn` (not `npm`) for frontend. Frontend root is `openmetadata-ui/src/main/resources/ui/`.
- **Docker services**: Development services (MySQL, Elasticsearch, etc.) run via `docker/development/docker-compose.yml`:
  ```bash
  docker compose -f docker/development/docker-compose.yml up -d
  ```

## Essential Development Commands

### Prerequisites and Setup
```bash
make prerequisites              # Check system requirements
source env/bin/activate         # ALWAYS activate venv first
cd ingestion && make install_dev_env  # Install Python dev dependencies
make generate                  # Generate Pydantic models from JSON schemas
make yarn_install_cache        # Install UI dependencies
```

### Frontend Development
```bash
cd openmetadata-ui/src/main/resources/ui
yarn start                     # Start development server on localhost:3000
yarn test                      # Run Jest unit tests
yarn test path/to/test.spec.ts # Run a specific test file
yarn test:watch               # Run tests in watch mode
yarn playwright:run            # Run E2E tests
yarn lint                      # ESLint check
yarn lint:fix                  # ESLint with auto-fix
yarn build                     # Production build
```

### Frontend CI Checkstyle (run before PR to match CI)
```bash
cd openmetadata-ui/src/main/resources/ui
yarn ui-checkstyle:changed         # One-shot checkstyle for changed files (excludes tsc)
yarn organize-imports:cli <files>  # Sort and organize imports
yarn lint:fix                      # ESLint auto-fix
yarn pretty:base --write <files>   # Prettier formatting
yarn license-header-fix <files>    # Add Apache 2.0 license headers
yarn i18n                          # Sync all 17 locale files with en-us.json
yarn generate:app-docs             # Regenerate application documentation
npx tsc --noEmit                   # TypeScript type check (catches errors early)
```

### Backend Development
```bash
mvn clean package -DskipTests  # Build without tests
mvn clean package -DonlyBackend -pl !openmetadata-ui  # Backend only
mvn test                       # Run unit tests
mvn verify                     # Run integration tests
mvn spotless:apply             # Format Java code
```

### Python Ingestion Development
```bash
cd ingestion
make install_dev_env           # Install in development mode
make generate                  # Generate Pydantic models from JSON schemas
make unit_ingestion_dev_env    # Run unit tests
make py_format                 # Apply ruff lint-fix + format
make py_format_check           # Verify lint + format (matches CI; catches non-auto-fixable issues)
make static-checks             # Run type checking with basedpyright
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-metadata/OpenMetadata](https://github.com/open-metadata/OpenMetadata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

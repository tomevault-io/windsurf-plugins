---
trigger: always_on
description: This file (symlinked as `CLAUDE.md`, `AGENTS.md`, and `.gemini/styleguide.md`) provides context and guidelines for AI agents working on the MCP Toolbox for Databases project. It summarizes key information from `CONTRIBUTING.md` and `DEVELOPER.md`.
---

# MCP Toolbox Context & Style Guide

This file (symlinked as `CLAUDE.md`, `AGENTS.md`, and `.gemini/styleguide.md`) provides context and guidelines for AI agents working on the MCP Toolbox for Databases project. It summarizes key information from `CONTRIBUTING.md` and `DEVELOPER.md`.

## Project Overview

**MCP Toolbox for Databases** is a Go-based project designed to provide Model Context Protocol (MCP) tools for various data sources and services. It allows Large Language Models (LLMs) to interact with databases and other tools safely and efficiently.


## Tech Stack

-   **Language:** Go (1.23+)
-   **Documentation:** Hugo (Extended Edition v0.146.0+)
-   **Containerization:** Docker
-   **CI/CD:** GitHub Actions, Google Cloud Build
-   **Linting:** `golangci-lint`

## Key Directories

-   `cmd/`: Application entry points.
-   `internal/sources/`: Implementations of database sources (e.g., Postgres, BigQuery).
-   `internal/tools/`: Implementations of specific tools for each source.
-   `tests/`: Integration tests.
-   `docs/en`: Project documentation. Separated logically into:
    - `documentation/`: Documentation and concepts (Section I).
    - `integrations/`: Reference architectures for DB connectivity and tools (Section II).
    - `samples/`: Tutorials and code samples (Section III).
    - `reference/`: CLI info and FAQs (Section IV).

## Development Workflow

### Prerequisites

-   Go 1.23 or later.
-   Docker (for building container images and running some tests).
-   Access to necessary Google Cloud resources for integration testing (if applicable).

### Building and Running

1.  **Build Binary:** `go build -o toolbox`
2.  **Run Server:** `go run .` (Listens on port 5000 by default)
3.  **Run with Help:** `go run . --help`
4.  **Test Endpoint:** `curl http://127.0.0.1:5000`

### Testing

-   **Unit Tests:** `go test -race -v ./cmd/... ./internal/...`
-   **Integration Tests:**
    -   Run specific source tests: `go test -race -v ./tests/<source_dir>`
    -   Example: `go test -race -v ./tests/alloydbpg`
    -   Add new sources to `.ci/integration.cloudbuild.yaml`
-   **Linting:** `golangci-lint run --fix`


## Developing Documentation

### Prerequisites

-   Hugo (Extended Edition v0.146.0+)
-   Node.js (for `npm ci`)

### Running Local Server

1.  Navigate to `.hugo` directory: `cd .hugo`
2.  Install dependencies: `npm ci`
3.  **Generate Search Index:** Because Pagefind requires physical files, `hugo server` alone will not populate the search bar. Build the local index first (using the development environment to block analytics) by running:
    `hugo --environment development && npx pagefind --site public --output-path static/pagefind`
4.  Start server: `hugo server`

### Versioning Workflows

Documentation builds automatically generate standard HTML alongside AI-friendly text files (`llms.txt` and `llms-full.txt`).

There are 6 workflows in total, handling parallel deployments to both GitHub Pages and Cloudflare Pages. **All deployment workflows automatically execute `npx pagefind --site public` to generate version-scoped search indexes.**

1.  **Deploy In-development docs**: Commits merged to `main` deploy to the `/dev/` path. Automatically defaults to version `Dev`.
2.  **Deploy Versioned Docs**: New GitHub releases deploy to `/<version>/` and the root path. The release tag is automatically injected into the build as the documentation version. *(Note: Developers must manually add the new version to the `[[params.versions]]` dropdown array in `hugo.toml` prior to merging a release PR).*
3.  **Deploy Previous Version Docs**: A manual workflow to rebuild older versions by explicitly passing the target tag via the GitHub Actions UI.

## Coding Conventions

### Tool Naming

-   **Tool Name:** `snake_case` (e.g., `list_collections`, `run_query`).
    -   Do *not* include the product name (e.g., avoid `firestore_list_collections`).
-   **Tool Type:** `kebab-case` (e.g., `firestore-list-collections`).
    -   *Must* include the product name.

### Branching and Commits

-   **Branch Naming:** `feat/`, `fix/`, `docs/`, `chore/` (e.g., `feat/add-gemini-md`).
-   **Commit Messages:** [Conventional Commits](https://www.conventionalcommits.org/) format.
    -   Format: `<type>(<scope>): <description>`
    -   Example: `feat(source/postgres): add new connection option`
    -   Types: `feat`, `fix`, `docs`, `chore`, `test`, `ci`, `refactor`, `revert`, `style`.

 ### PR Title Format

Format: `<type>[optional scope]: <description>`

- **Example:** `feat(source/postgres): add support for "new-field" field`
- **Example (Breaking Change):** `fix(tool/sql)!: change default parameter value`

#### Types

| Type | Description | Version change affected |
| :--- | :--- | :--- |
| **BREAKING CHANGE** | Anything with this type or a `!` after the type/scope introduces a breaking API change. E.g. `fix!: description` or `feat!: description`. | major |
| **feat** | Adding a new feature to the codebase. | minor |
| **fix** | Fixing a bug or typo in the codebase. | patch |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [googleapis/mcp-toolbox](https://github.com/googleapis/mcp-toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Pre-commit Verification

Always run `make lint` before considering changes complete. The repository has a husky pre-commit hook that runs `pnpm format:check`, `pnpm lint`, and `pnpm typecheck:src`. All three must pass before committing.

Always verify the Docker build works by running `docker build -t docsearch-mcp:test .` after making changes to build configuration, TypeScript config, dependencies, or Dockerfile-related files.

## Project Overview

This is a local-first document search and indexing system with both a CLI tool and MCP server that provides hybrid semantic+keyword search across local files (including PDFs) and Confluence pages. The system chunks documents, creates embeddings, stores them in SQLite with vector search capabilities, and exposes search functionality through both a command-line interface and the Model Context Protocol.

## Development Commands

```bash
# Setup
make setup                   # Install dependencies and setup .env
make install                 # Install dependencies only
pnpm i                       # Install dependencies only (alternative)
cp .env.example .env         # Set up environment variables

# Development Servers
make dev                     # Start MCP development server
make dev-cli                 # Start CLI in development mode
pnpm dev:cli                 # Start CLI in development mode (alternative)
pnpm dev:mcp                 # Start MCP server in development (alternative)

# Build Commands
make build                   # Build the project
make clean                   # Clean all generated files (node_modules, data, dist)
make clean-dist              # Clean build directory only
pnpm build                   # Build TypeScript (alternative)

# Production
make start                   # Start production MCP server
make start-cli               # Start CLI in production mode
pnpm start:mcp               # Start built MCP server (alternative)
pnpm start:cli               # Run built CLI (alternative)

# Quality Assurance
make test                    # Run tests
make test-run                # Run tests once
make test-ui                 # Run tests with UI
make test-coverage           # Run tests with coverage
make test-unit               # Run unit tests only
make test-integration        # Run integration tests (requires Docker)
make lint                    # Run linter, formatter, and typecheck (source only)
make lint-fix                # Run linter with auto-fix
make format                  # Format code with Prettier
make format-check            # Check code formatting
make typecheck               # Run TypeScript type checking (all files)
make typecheck-src           # Run TypeScript type checking (source only)
make check-all               # Run all quality checks (lint, typecheck, unit tests)

# Data Management
make ingest-files            # Ingest local files
make ingest-files-incremental # Ingest local files with incremental indexing
make ingest-confluence       # Ingest Confluence pages
make ingest-all              # Ingest all sources (files and confluence)
make ingest-all-incremental  # Ingest all sources with incremental indexing
make watch                   # Watch for file changes and re-index
make watch-incremental       # Watch for file changes with incremental re-indexing
make search QUERY="text"     # Search documents
make search-json QUERY="text" # Search documents with JSON output
make clean-data              # Clean data directory

# Incremental Indexing (Performance Optimized)
make incremental-files       # Alias for incremental file indexing
make incremental-all         # Alias for incremental indexing of all sources
make incremental-watch       # Alias for incremental file watching
make incremental-benchmark   # Compare full vs incremental indexing performance

# Ansible Deployment (deploy/ansible/)
make ansible-test            # Run Molecule tests for Ansible deployment (requires Docker)
make ansible-lint            # Lint Ansible playbook with ansible-lint

# Alternative pnpm commands
pnpm test                    # Run tests in watch mode
pnpm test:run                # Run tests once
pnpm test:ui                 # Run tests with UI
pnpm test:coverage           # Run tests with coverage
pnpm lint                    # Run ESLint
pnpm lint:fix                # Run ESLint with auto-fix
pnpm format                  # Format code with Prettier
pnpm typecheck               # Run TypeScript type checking

# CLI Tool (direct pnpm usage)
pnpm dev:cli ingest files    # Index local files via CLI
pnpm dev:cli ingest confluence # Index Confluence pages via CLI
pnpm dev:cli ingest all --watch # Index all sources with file watching
pnpm dev:cli search "query"  # Search documents via CLI
pnpm dev:cli search "test" --output json # Search with JSON output

# Help
make help                    # Show all available make commands
```

## Architecture

### Core Components

- **CLI Tool** (`src/cli/`): Command-line interface with ports and adapters architecture
  - Domain layer with clean interfaces (`src/cli/domain/ports.ts`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PatrickKoss/docsearch-mcp](https://github.com/PatrickKoss/docsearch-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

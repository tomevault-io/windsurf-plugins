---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

Xget is a high-performance, Cloudflare Workers-based acceleration engine for
developer resources. It provides unified acceleration for code repositories
(GitHub, GitLab, etc.), package registries (npm, PyPI, Maven, etc.), container
registries (Docker Hub, GHCR, etc.), and AI inference APIs (OpenAI, Anthropic,
etc.).

The project operates as a reverse proxy that transforms incoming requests to
match various platform APIs while adding security headers, caching, retry logic,
and performance monitoring.

## Development Commands

### Core Commands

```bash
# Start development server (Cloudflare Workers local environment)
npm run dev              # Runs on http://localhost:8787

# Deploy to Cloudflare Workers production
npm run deploy

# Build and run tests
npm run test             # Run tests in watch mode
npm run test:run         # Run tests once
npm run test:coverage    # Generate coverage report
npm run test:ui          # Open Vitest UI

# Code quality
npm run lint             # Check code quality
npm run lint:fix         # Fix linting issues
npm run format           # Format code with Prettier
npm run format:check     # Check formatting without changes
npm run type-check       # TypeScript type checking (no emit)
npm run commitlint       # Validate the latest commit message
```

## Commit Messages

- Use [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) for
  every commit
- Preferred format: `type(scope): description`
- Common types: `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`
- The repository installs a `commit-msg` hook via `npm install`; do not bypass
  it unless explicitly required

## Pre-Commit Requirements

- Before every commit, run the local CI-equivalent checks from
  `.github/workflows/ci.yml`
- Required commands: `npm run lint`, `npm run format:check`, `npm run test:run`,
  and `npm run type-check`
- If any required check fails, do not commit until the failure is resolved
- Apply this rule to every commit, including documentation-only changes, unless
  the user explicitly asks for a different workflow

### Testing Workflow

- Tests use Vitest with `@cloudflare/vitest-pool-workers` for Workers-specific
  testing
- Run `npm run test:run` before committing to ensure all tests pass
- Coverage reports are generated in `coverage/` directory

## Architecture

### Request Flow

1. **Entry Point**: `src/index.js` - Exports default Worker with `fetch()`
   handler
2. **Validation**: `src/utils/validation.js` - Validates HTTP methods, path
   length, detects protocol types
3. **Platform Detection**: URL path is parsed to identify platform (e.g., `/gh/`
   → GitHub)
4. **Path Transformation**:
   `src/routing/platform-transformers.js#transformPath()` converts request paths
   to upstream URLs
5. **Protocol Handling**: Different handlers for Git, Docker, AI inference
   requests
6. **Upstream Fetch**: Request forwarded with appropriate headers and retry
   logic
7. **Response Processing**: URL rewriting for certain platforms (npm, PyPI),
   cache storage
8. **Security Headers**: Added via `src/utils/security.js` before returning to
   client

### Key Components

#### Configuration (`src/config/`)

- **`index.js`**: Runtime configuration with environment variable overrides
  - `TIMEOUT_SECONDS`: Request timeout (default: 30s)
  - `MAX_RETRIES`: Retry attempts (default: 3)
  - `CACHE_DURATION`: Fallback mutable cache TTL (default: 300s = 5 minutes)
  - `SECURITY.ALLOWED_METHODS`: HTTP methods (default: GET, HEAD)

- **`platform-catalog.js`**: Platform base URL definitions
  - `PLATFORM_CATALOG`: Object mapping platform keys to base URLs

- **`routing/platform-index.js`**: Pre-sorted keys for efficient matching
  - `SORTED_PLATFORMS`: Longest-prefix-first platform matching order

- **`routing/platform-transformers.js`**: Platform-specific path rewriting
  - `transformPath()`: Converts request paths to platform-specific URLs
  - Special handling for crates.io (adds `/api/v1/crates` prefix) and Jenkins
    (adds `/current/` prefix)

#### Protocol Handlers (`src/protocols/`)

- **`git.js`**: Git protocol detection and header configuration
  - Detects Git operations via User-Agent, endpoints (`/info/refs`,
    `/git-upload-pack`)
  - Handles Git LFS via `Accept: application/vnd.git-lfs+json`

- **`docker.js`**: Container registry protocol (OCI/Docker)
  - Parses WWW-Authenticate headers for token authentication
  - Handles Docker registry v2 API authentication flow
  - Special redirect handling to prevent leaking auth tokens to blob storage

- **`ai.js`**: AI inference API detection and header forwarding
  - Detects requests to `/ip/*` platforms
  - Preserves all headers for AI API compatibility

#### Utilities (`src/utils/`)

- **`validation.js`**: Request validation logic
  - `isDockerRequest()`: Detects Docker/OCI operations
  - `validateRequest()`: Enforces security policies

- **`security.js`**: Security headers and error responses
  - Adds HSTS, X-Frame-Options, CSP, X-XSS-Protection
  - `createErrorResponse()`: Generates standardized error responses

- **`performance.js`**: Performance monitoring

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xixu-me/xget](https://github.com/xixu-me/xget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->

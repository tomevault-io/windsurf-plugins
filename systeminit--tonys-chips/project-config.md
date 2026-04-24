---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is "Tony's World of Chips", an e-commerce storefront application built as a monorepo with npm workspaces. The application sells potato chips and demonstrates a full-stack TypeScript application with Docker containerization.

## Monorepo Structure

The project uses npm workspaces with two packages:
- `packages/api/` - Express.js REST API with Prisma ORM
- `packages/web/` - Express.js server-side rendered frontend with EJS templates
- `e2e/` - Playwright end-to-end tests (in root directory)

The root `tsconfig.json` is shared by both packages. Each package extends it with package-specific settings.

## Development Commands

### Running Locally

```bash
# API (from packages/api/)
npm run dev          # Start development server on port 3000
npm run build        # Build TypeScript
npm start            # Run production build
npm test             # Run all tests with Jest
npm run test:watch   # Watch mode
npm run test:coverage # With coverage report

# Web (from packages/web/)
npm run dev          # Start development server on port 3001
npm run build        # Build TypeScript
npm start            # Run production build
npm run lint         # Run ESLint

# E2E Tests (from root directory)
npm run test:e2e     # Run Playwright tests
npm run test:e2e:ui  # Run tests with UI
npm run test:e2e:report # Show test report
```

### Database Commands (from packages/api/)

```bash
npx prisma migrate dev --name <name>  # Create and apply migration
npx prisma migrate deploy              # Apply migrations in production
npx prisma db seed                     # Seed database
npx prisma studio                      # Open Prisma Studio GUI
npx prisma generate                    # Generate Prisma Client
```

### Docker Commands

```bash
# Local build commands (uses CI orchestration)
npm run build:local          # Build API, web, and E2E images with latest tag

# CI npm scripts (for use in GitHub Actions or manual deployment)
npm run ci:manage-image-lifecycle  # Unified image lifecycle management

# Or use CI commands directly for granular control
npx tsx ci/main.ts build-local all    # Build all images (api + web + e2e)
npx tsx ci/main.ts build-local api    # Build only API image
npx tsx ci/main.ts build-local web    # Build only web image
npx tsx ci/main.ts build-local e2e    # Build only E2E test image

# Image Lifecycle Management Commands (CI/Deployment with specific tags)
npx tsx ci/main.ts manage-image-lifecycle build <environment> <component> <tag>    # Build only
npx tsx ci/main.ts manage-image-lifecycle publish <environment> <component> <tag>  # Publish only
npx tsx ci/main.ts manage-image-lifecycle deploy <environment> <tag>   # Deploy all components

# Docker Compose orchestration (direct docker-compose commands)
npm run docker:up            # Start services using latest tag (foreground)
npm run docker:down          # Stop and remove containers
npm run docker:logs          # View logs for all services

# Or use docker-compose directly
docker-compose up            # Start services (foreground)
docker-compose up -d         # Start services (background)
docker-compose down          # Stop and remove containers
docker-compose logs -f       # View all logs
docker-compose logs -f api   # View API logs only

# Use specific image tag (for remote deployment or versioned testing)
IMAGE_TAG=20250115120000-abc123 docker-compose up
IMAGE_TAG=20250115120000-abc123 docker-compose up -d
```

### E2E Testing with Docker

The E2E tests can be run in a Docker container for consistent execution across environments:

```bash
# Build all images including E2E (local development)
npm run build:local
# Or build E2E only: npx tsx ci/main.ts build-local e2e

# Build E2E image for CI/deployment with specific tag
npx tsx ci/main.ts build sandbox e2e 20250115120000-abc123

# Run E2E tests against local docker-compose services
npm run docker:test:e2e
# Or: npx tsx ci/main.ts test-e2e

# Run against remote services by setting environment variables
API_URL=https://api.example.com WEB_URL=https://www.example.com npm run docker:test:e2e
# Or: API_URL=https://api.example.com WEB_URL=https://www.example.com npx tsx ci/main.ts test-e2e

# Run directly with docker for custom options (save test results)
docker run --rm \
  -e API_URL=https://api.example.com \
  -e WEB_URL=https://www.example.com \
  -v $(pwd)/test-results:/app/test-results \
  -v $(pwd)/playwright-report:/app/playwright-report \
  tonys-chips/e2e:latest
```

**Environment Variables for E2E Container**:
- `API_URL` - Base URL for the API (default: http://localhost:3000)
- `WEB_URL` - Base URL for the web application (default: http://localhost:8080)
- `CI` - Set to "true" to enable CI mode (enabled by default in container)

### Policy Checking

The project includes an integrated policy compliance checker that evaluates System Initiative infrastructure against defined policies using Claude AI.

**Policy Structure**: Policy files are markdown documents in the `policy/` directory with a specific format:

```markdown
# Policy Title

## Policy
Policy description and requirements...

### Exceptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/systeminit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

---
trigger: always_on
description: This is a ChatGPT + Enterprise data application built with Azure OpenAI and Azure AI Search, implementing the Retrieval Augmented Generation (RAG) pattern. The application allows users to chat with their enterprise data using Azure OpenAI's ChatGPT models (gpt-4o-mini) and Azure AI Search for data indexing and retrieval.
---

# AGENTS.md

## Project Overview

This is a ChatGPT + Enterprise data application built with Azure OpenAI and Azure AI Search, implementing the Retrieval Augmented Generation (RAG) pattern. The application allows users to chat with their enterprise data using Azure OpenAI's ChatGPT models (gpt-4o-mini) and Azure AI Search for data indexing and retrieval.

**Architecture**: The project is organized as an npm workspace with three main packages:
- **webapp**: React + Vite frontend application with Lit web components (Static Web App)
- **search**: Fastify-based backend API service for search and chat (Container App)
- **indexer**: Document indexing service with CLI tool (Container App)

**Key Technologies**:
- TypeScript, React, Lit (web components)
- Fastify (backend API framework)
- Vite (frontend build tool)
- Azure OpenAI, Azure AI Search, Azure Container Apps, Azure Static Web Apps
- Playwright (e2e testing), k6 (load testing)
- Azure Developer CLI (azd) for infrastructure and deployment

## Setup Commands

### Prerequisites
- Node.js 22 or higher (check `.nvmrc`)
- npm 10 or higher
- Azure Developer CLI (azd) - for deployment
- Azure subscription with OpenAI service access

### Installation

```bash
# Install all dependencies (workspace root)
npm install

# Install Playwright browsers (for e2e tests)
npx playwright install --with-deps
```

### First-Time Azure Deployment

```bash
# Login to Azure
azd auth login

# Deploy infrastructure and application
azd up

# This will:
# - Prompt for Azure location
# - Provision all Azure resources (OpenAI, AI Search, Container Apps, etc.)
# - Build and deploy all services
# - Run post-provision hook to index sample data
```

## Development Workflow

### Running Locally

**Prerequisites**: You must first deploy to Azure with `azd up` before running locally.

```bash
# 1. Authenticate with Azure
azd auth login
az login

# 2. Load environment variables from Azure deployment
azd env get-values > .env

# 3. Index the data
./scripts/index-data.sh    # On Linux/macOS
./scripts/index-data.ps1   # On Windows

# 4. Start all services concurrently
npm start

# This runs all three services:
# - webapp on http://localhost:5173
# - search API on http://localhost:3000
# - indexer API on http://localhost:3001
```

### Starting Individual Services

```bash
# Start only the webapp
npm run start:webapp

# Start only the search API
npm run start:search

# Start only the indexer API
npm run start:indexer
```

### Development Mode with Hot Reload

Each workspace package has its own dev mode with hot reload:

```bash
# Build and watch for changes in a specific package
npm run dev --workspace=webapp
npm run dev --workspace=search
npm run dev --workspace=indexer
```

### Environment Variables

Environment variables are managed through Azure Developer CLI:

```bash
# View all environment variables
azd env get-values

# Set a specific environment variable
azd env set VARIABLE_NAME value

# Export to .env file for local development
azd env get-values > .env
```

## Testing Instructions

### Unit Tests

```bash
# Run all tests across workspaces
npm test

# Run tests for a specific package
npm test --workspace=search
npm test --workspace=indexer

# Tests use Node.js built-in test runner and c8 for coverage
```

### End-to-End Tests (Playwright)

```bash
# Run Playwright e2e tests
npm run test:playwright

# Run in headed mode for debugging
npx playwright test --headed

# Run specific test file
npx playwright test tests/e2e/webapp.spec.ts

# View test report
npx playwright show-report
```

**Test Configuration**: See `playwright.config.ts`
- Tests run against `http://localhost:5173`
- Uses chromium, firefox, and webkit browsers
- Automatically starts webapp dev server before tests
- Test files located in `tests/e2e/`

### Load Tests

```bash
# Run load tests with k6
npm run test:load

# Tests are defined in tests/load/index.js
```

### Test File Locations

- Unit tests: `packages/*/test/**/*.ts`
- E2E tests: `tests/e2e/**/*.spec.ts`
- Load tests: `tests/load/**/*.js`

## Code Style

### Linting and Formatting

```bash
# Run ESLint
npm run lint

# Fix ESLint issues automatically
npm run lint:fix

# Check code formatting with Prettier
npm run format:check

# Format code with Prettier
npm run format
```

### Style Conventions

**Prettier Configuration** (in `package.json`):
- Tab width: 2 spaces
- Semicolons: required
- Single quotes: enabled
- Print width: 120 characters
- Bracket spacing: enabled

**ESLint**: Uses shared config from `eslint-config-shared` workspace package

**Pre-commit Hooks**: 
- Uses `simple-git-hooks` with `lint-staged`
- Automatically runs on `git commit`
- Formats staged files with Prettier

### File Organization

```
/
├── packages/           # Workspace packages
│   ├── webapp/        # Frontend application
│   ├── search/        # Search API service
│   ├── indexer/       # Indexer service
│   ├── chat-component/# Shared chat web component
│   └── eslint-config/ # Shared ESLint config

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/azure-search-openai-javascript](https://github.com/Azure-Samples/azure-search-openai-javascript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

---
trigger: always_on
description: This is an OpenCode plugin project for querying Z.ai GLM Coding Plan usage statistics.
---

# AGENTS.md
This is an OpenCode plugin project for querying Z.ai GLM Coding Plan usage statistics.

**Current Phase:** Implementation
**Status:** PRD finalized, ready for implementation
**Architecture:** OpenCode Plugin (not standalone CLI script)

This document provides build commands, code style guidelines, and conventions for agentic coding agents working on this OpenCode plugin repository.

## Build & Development Commands

```bash
# Build TypeScript to JavaScript
npm run build

# Clean build artifacts
npm run clean

# Run all tests
npm run test

# Run specific test file
npm run test -- path/to/test.test.ts

# Watch mode during development (if configured)
npm run test -- --watch

# Lint source code
npm run lint

# Prepare for npm publish
npm run prepublishOnly
```

## GitHub Actions CI/CD

This project uses GitHub Actions for automated CI/CD.

### CI Workflow (Automatic)

**Triggers:**
- Push to `main` branch
- Pull requests to `main` branch

**What It Does:**
- Tests code on multiple Node.js versions (18.x, 20.x, 22.x)
- Runs linting (`npm run lint`)
- Builds TypeScript (`npm run build`)
- Runs all tests (`npm test`)
- Validates documentation links
- Uploads test coverage artifacts

**View Results:**
- Check the "Actions" tab in GitHub repository
- Green checkmark ✅ = All tests passed
- Red X ❌ = Something failed (check logs)

### Publish Workflow (On Release)

**Triggers:**
- GitHub release published

**What It Does:**
- Builds, tests, and lints code
- Publishes package to npm with provenance
- Publishes package to GitHub Packages as scoped package (`@<repo-owner>/opencode-glm-quota`)
- Creates release artifacts

**Setup Required:**
1. Create npm automation token at https://www.npmjs.com/settings/tokens
2. Add `NPM_TOKEN` to GitHub repository secrets:
   - Go to Settings → Secrets and variables → Actions
   - Create secret: `NPM_TOKEN`
   - Value: Your npm automation token

**How to Publish:**
```bash
# Using GitHub CLI
gh release create v1.0.0 --generate-notes

# Or via GitHub web UI:
# 1. Go to Releases → Create new release
# 2. Enter version tag (e.g., v1.0.0)
# 3. Add release notes
# 4. Click "Publish release"
```

The workflow automatically publishes to npm and GitHub Packages when a release is created.

### CI Status Badge

The README includes a CI status badge that shows the current build status:
```
[![Build Status](https://github.com/guyinwonder168/opencode-glm-quota/workflows/CI/badge.svg)]
```

This badge updates automatically with each commit.

## Project Structure

```
src/
  index.ts           # Main plugin entry point
  api/
    client.ts        # HTTPS client with timeout and error handling
    endpoints.ts     # Platform-specific API endpoints
    platforms.ts     # Platform detection and naming
  utils/
    date-formatter.ts # Date/time formatting utilities
    progress-bar.ts   # ASCII progress bar rendering
    time-window.ts    # Rolling window calculation
  integration/
    agents/glm-quota-exec.md     # Minimal executor agent (Markdown)
    command/glm_quota.md         # /glm_quota slash command
    skills/glm-quota/SKILL.md     # Skill documentation (OpenCode spec: skills/<name>/SKILL.md)
  bin/
    install.js                    # Installation script
  dist/                             # Compiled JavaScript (generated)
  tests/                            # Test suite
  package.json                       # Dependencies and scripts
  tsconfig.json                      # TypeScript configuration
```

## Code Style Guidelines

### TypeScript Configuration
- Target: ES2022
- Module: NodeNext
- Strict mode enabled
- Always use type annotations for function returns
- Use `as const` for constants that shouldn't be modified

### Import Order
1. Core Node.js modules (`fs`, `path`, `os`, `https`)
2. Third-party imports (`@opencode-ai/plugin`)
3. Local imports (if any)
4. Type imports (use `import type` where possible)

```typescript
// ✅ Correct
import * as fs from "fs"
import * as path from "path"
import { type Plugin, tool } from "@opencode-ai/plugin"
```

### Naming Conventions
- **Constants**: `UPPER_SNAKE_CASE` for immutable constants
- **Functions**: `camelCase` for regular functions
- **Types/Interfaces**: `PascalCase` (e.g., `ApiResponse`, `Credentials`)
- **Interfaces**: Describe data structures; use for object shapes
- **Type Aliases**: Use for unions, literals, or primitives

```typescript
const ENDPOINTS = { /* ... */ }  // UPPER_SNAKE_CASE
function getCredentials() { }     // camelCase
interface ApiResponse { }         // PascalCase
type Platform = 'ZAI' | 'ZHIPU'   // PascalCase for union types
```

### Error Handling
- Always wrap file operations in try-catch blocks
- Use `null` returns for optional values, not errors
- Include fallback mechanisms (e.g., auth.json → env vars)
- Return user-friendly error messages with setup instructions

```typescript
async function getCredentials(): Promise<Credentials | null> {
  try {
    // Primary method
  } catch {
    // Silent fail, try next method
  }
  // Fallback methods
  return null  // No credentials found
}
```

### API & HTTP Requests
- Use native `https` module (no fetch for Node.js compatibility)
- **CRITICAL**: Do NOT use "Bearer" prefix in Authorization header

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guyinwonder168/opencode-glm-quota](https://github.com/guyinwonder168/opencode-glm-quota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

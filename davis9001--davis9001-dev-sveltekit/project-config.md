---
trigger: always_on
description: **Dev Server is Always Running (Local Copilot Chat Only)**
---

# GitHub Copilot Instructions for davis9001.dev

## 🖥️ Development Environment Assumptions

**Dev Server is Always Running (Local Copilot Chat Only)**

> ⚠️ **This section applies ONLY to GitHub Copilot Chat on local workstations, NOT to the Copilot Coding Agent (remote/cloud agent).**

When using Copilot Chat locally:

- Assume `npm run dev` is already running in a separate terminal on port 4220
- Do NOT start the dev server when performing tasks
- Do NOT run `npm run dev`, `vite dev`, or similar commands
- When testing locally, assume the app is already accessible at `http://localhost:4220`
- If you need to verify the app is running, check the existing terminal output rather than starting a new instance

When using the Copilot Coding Agent (remote):

- The coding agent should manage its own dev server as needed
- Normal startup commands are expected in that context

## 🎯 Core Development Philosophy

**Test-Driven Development (TDD) is MANDATORY**

- Never start a feature or bug fix without writing tests first
- Write failing tests, then implement code to make them pass
- Maintain 95%+ code coverage across all modules — coverage must NEVER drop below 95%
- Tests are not optional—they are part of the definition of "done"

## 🏗️ Architecture Principles

### Cloudflare-First Development

- **Always optimize for Cloudflare Workers runtime**
- Use Cloudflare services as the default choice:
  - D1 for database operations
  - KV for key-value storage
  - R2 for object storage
  - Queues for background jobs
  - Turnstile for CAPTCHA
  - Workers AI for AI/ML operations
- Avoid external services that duplicate Cloudflare functionality
- Consider edge computing patterns and cold start optimization

### Minimal External Dependencies

- **Build, don't buy** - Implement features in-house whenever feasible
- Avoid external packages for:
  - WYSIWYG editors (build custom)
  - User management (use built-in auth)
  - SSO integrations (implement directly with Auth.js)
  - UI components (extend internal component library)
  - Form validation (custom implementations)
  - State management (use Svelte stores)
- Only add external packages when:
  - The functionality is extremely complex (e.g., cryptography)
  - It's a Cloudflare-native integration
  - It's a core framework requirement (Svelte, Vite)
  - The package is well-maintained, lightweight, and has no alternatives

## 🧪 Testing Standards

### Test Structure

```typescript
// tests/unit/feature.test.ts
import { describe, it, expect, beforeEach, afterEach } from 'vitest';

describe('Feature Name', () => {
	beforeEach(() => {
		// Setup test environment
	});

	afterEach(() => {
		// Cleanup
	});

	it('should do X when Y happens', () => {
		// Arrange
		// Act
		// Assert
	});
});
```

### Coverage Requirements

- **Minimum 95% coverage** on all modules — this is a hard floor, never allow it to drop below 95%
- 100% coverage on critical paths (auth, payments, data mutations)
- **Before finishing any task**, run `npm run test:coverage` and verify coverage has not decreased
- If your changes would reduce coverage below 95%, you MUST add additional tests before considering the task complete
- Every new feature must include:
  - Unit tests for business logic
  - Integration tests for API endpoints
  - Component tests for UI elements
  - E2E tests for critical user flows

### Test Types

1. **Unit Tests** (`tests/unit/`)
   - Pure functions and utilities
   - Individual Svelte components
   - Store logic
   - Database queries (mocked)

2. **Integration Tests** (`tests/integration/`)
   - API endpoints
   - Database operations with test D1 instance
   - Service layer interactions

3. **E2E Tests** (`tests/e2e/`)
   - Complete user workflows
   - Authentication flows
   - Critical business processes

## 🚀 Development Workflow

### 1. Starting Any Work

```bash
# Always ensure dev environment works
npm run dev

# Create feature branch
git checkout -b feature/feature-name

# Write tests FIRST
# Create test file before implementation file
```

### 2. TDD Cycle (Red-Green-Refactor)

1. **RED**: Write a failing test that defines desired behavior
2. **GREEN**: Write minimal code to make the test pass
3. **REFACTOR**: Improve code quality while keeping tests green
4. **REPEAT**: Continue for each small piece of functionality

### 3. Before Committing

```bash
# Run all tests
npm run test

# Check coverage
npm run test:coverage

# Type checking
npm run check

# Ensure dev still works
npm run dev
```

## 📁 Project Structure

```
davis9001.dev/
├── src/
│   ├── lib/
│   │   ├── components/      # Svelte components (with .test.ts files)
│   │   ├── stores/          # Svelte stores (with .test.ts files)
│   │   ├── utils/           # Utility functions (with .test.ts files)
│   │   ├── services/        # Business logic (with .test.ts files)
│   │   └── types/           # TypeScript types
│   ├── routes/              # Application routes (with .test.ts files)
│   └── app.html
├── tests/
│   ├── unit/                # Unit tests
│   ├── integration/         # Integration tests
│   ├── e2e/                 # End-to-end tests
│   └── fixtures/            # Test data and mocks
├── migrations/              # D1 database migrations
└── wrangler.toml           # Cloudflare configuration
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davis9001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

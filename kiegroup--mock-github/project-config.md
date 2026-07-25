---
trigger: always_on
description: **mock-github** is a testing library that provides tools to mock GitHub environments locally. It enables testing GitHub Actions and API calls without polluting real repositories or hitting rate limits.
---

# AI Coding Agent Instructions for mock-github

## Project Overview
**mock-github** is a testing library that provides tools to mock GitHub environments locally. It enables testing GitHub Actions and API calls without polluting real repositories or hitting rate limits.

Two core abstractions:
1. **Moctokit**: Mocks GitHub REST API endpoints using OpenAPI spec (via `nock`)
2. **MockGithub**: Mocks full GitHub environment with repositories, branches, files, actions, and env vars

## Architecture & Key Components

### Layer 1: Endpoint Mocking (Foundation)
- `src/endpoint-mocker/abstract-endpoint-mocker.ts`: Base class using `nock` for HTTP interception
- `src/endpoint-mocker/request/abstract-request-mocker.ts`: Generic request parameter matching
- `src/endpoint-mocker/response/abstract-response-mocker.ts`: Response building with status/headers

### Layer 2: API Mocking (Generated)
- `src/moctokit/moctokit.ts`: Extends `EndpointMocker`, wraps auto-generated endpoint methods
- `src/moctokit/generated/endpoint-request.ts`: **AUTO-GENERATED** from GitHub OpenAPI spec (~1817 lines)
  - Maps 200+ GitHub API endpoints to `MoctokitRequestMocker` instances
  - **Never manually edit** - regenerate via `npm run update:endpoint-requests`
- `src/moctokit/generated/endpoint-details.ts`: OpenAPI metadata for each endpoint

### Layer 3: GitHub Environment Mocking
- `src/github/github-mocker.ts`: Orchestrates setup/teardown of repositories, env vars, actions
- `src/github/repository/repository-mocker.ts`: Creates local git repos via `simple-git`
- `src/github/env/env-mocker.ts`: Manages process environment variables
- `src/github/action/action-mocker.ts`: Simulates GitHub Action execution context

### Type System (Critical for Development)
- Uses **OpenAPI path types** from generated endpoint details
- Generic constraints like `Path extends keyof paths, Method extends keyof paths[Path]`
- `src/moctokit/response/response-mocker.types.ts`: Complex conditional types for response validation
- `.types.ts` files excluded from coverage - contain domain models, not business logic

## Critical Developer Workflows

### Build & Dependencies
```bash
npm run build        # TypeScript → JavaScript, alias resolution via tsc-alias
npm test             # Jest with 90% coverage threshold
npm run test:report  # Coverage report + SonarQube XML output
npm run lint         # ESLint with auto-fix on commit via husky
```

### Updating Generated Code
```bash
npm run update:api-spec           # Fetch latest GitHub OpenAPI spec
npm run update:endpoint-details   # Parse endpoints from spec
npm run update:endpoint-requests  # Generate endpoint method bindings
npm run update-all                # Run all three steps
```

Scripts are in `scripts/` directory - regenerate when GitHub API changes.

## Project-Specific Patterns

### 1. **Module Path Aliases**
- Use `@mg/*` imports (configured in `tsconfig.json` paths)
- Example: `import { Moctokit } from "@mg/moctokit/moctokit"`
- Resolves to `src/*` during development, `build/src/*` in compiled output

### 2. **Abstract Base Classes for Extension**
- `EndpointMocker`, `RequestMocker`, `ResponseMocker` are abstract - meant to be extended
- `MoctokitRequestMocker` and `MoctokitResponseMocker` extend these for GitHub API specifics
- Pattern: Abstract → Concrete implementation → Usage via interface

### 3. **Config Validation with AJV**
- `MockGithub` constructor validates config against `GithubConfigSchema` (JSON Schema)
- Schema defined in `src/github/schema/` directory
- Config can be JSON file path (string) or object - validated before use

### 4. **Setup/Teardown Lifecycle**
- `MockGithub` requires explicit `setup()` and `teardown()` calls
- Setup order matters: repositories → env vars → actions
- Throws if methods called before setup (e.g., `repo` getter checks `hasCalledSetup`)

### 5. **Repository State Management**
- `RepositoryState` tracks branches, files, remotes (origin/upstream)
- State exposed via `repo.getState()`, `repo.getBranchState()`, etc.
- Branches can be local or pushed - tracked separately in `BranchState`

### 6. **File Organization by Responsibility**
```
src/
├── endpoint-mocker/      # Generic HTTP mocking layer
├── moctokit/             # GitHub API-specific mocking
├── github/               # Full GitHub environment
│   ├── repository/       # Git repo + state management
│   ├── action/           # GitHub Actions simulation
│   ├── env/              # Environment variables
│   └── schema/           # JSON validation schemas
```

## Integration Points & Dependencies

### External Libraries
- `nock`: HTTP mocking (≥1.0.0 incompatible with Node 18 native fetch)
- `simple-git`: Git operations (local repo creation/manipulation)
- `@octokit/rest`: Type definitions (v19, not runtime)
- `ajv`: JSON Schema validation
- `ts-jest`: TypeScript testing

### Cross-Component Communication
1. **Config Flow**: JSON config → validated → distributed to Action/Env/Repo mockers
2. **Setup Coordination**: `MockGithub.setup()` → parallel repo/env setup → then action setup
3. **Query Interface**: `repo.getState()` returns `State` (path, branches, files)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kiegroup/mock-github](https://github.com/kiegroup/mock-github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

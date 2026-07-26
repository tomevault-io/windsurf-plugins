---
trigger: always_on
description: **Project purpose:** Lightweight Docker Swarm dashboard. Target image size < 25 MB. Not intended for public exposure. See `README.md` for details.
---

## Copilot / AI Agent Instructions for docker-swarm-dashboard

**Project purpose:** Lightweight Docker Swarm dashboard. Target image size < 25 MB. Not intended for public exposure. See `README.md` for details.

---

## 1. Project Overview

### Architecture
- React frontend (`app-src/`) + Go backend (`server-src/`)
- Frontend: React 19, Webpack, Jotai (`atomWithHash` for URL-hash state)
- Backend: Docker SDK and gorilla/websocket
- Mock API (`app-src/mock/api/api-mock.mjs`) runs on port 3001
- Dev server runs on port 3000

### Runtime Requirements
- **Node.js:** >= 22.0.0
- **Go:** 1.25
- **Package manager:** yarn (required, not npm)

### Key Entry Points
- `app-src/package.json` — all scripts and dependencies (use `yarn`)
- `app-src/src/common/store/atoms/index.js` — all Jotai atom definitions
- `server-src/main.go` — HTTP router and Docker client wiring
- `.github/workflows/` — CI; run `grep '^name:' .github/workflows/*.yml` for all workflow names

---

## 2. Developer Commands

### Frontend Development (app-src directory)
```bash
# Start development environment (mock API + dev server + open browser)
cd app-src && yarn start

# Start servers without opening browser
cd app-src && yarn start:servers

# Production build
cd app-src && yarn build

# Asset downloads (postinstall)
node download-files.js
```

### Backend Development (server-src directory)
```bash
# Production build
cd server-src && go build -o docker-swarm-dashboard

# Docker build (from root directory)
docker build -t ghcr.io/heckenmann/docker-swarm-dashboard:local .
```

### Testing

#### Frontend Unit Tests (app-src directory)
```bash
# Run all tests
cd app-src && yarn test

# Run with coverage
cd app-src && yarn test:coverage

# Run a single test file
cd app-src && yarn test tests/unit/path/to/test-file.test.js

# Run tests matching a pattern
cd app-src && yarn test --testNamePattern="pattern"

# Watch mode
cd app-src && yarn jest --watch
cd app-src && yarn jest --watch tests/unit/path/to/test-file.test.js
```

#### Frontend E2E Tests (app-src directory)
```bash
# Run all Cypress tests
cd app-src && yarn run cy:run --browser electron

# Run specific test
cd app-src && yarn run cy:run --spec cypress/e2e/<spec>.cy.js --browser electron

# Open Cypress UI
cd app-src && yarn cy:open

# Run in headless mode (CI)
cd app-src && yarn dev:cy:run
```

#### Backend Tests (server-src directory)
```bash
# Run all Go tests
cd server-src && go test ./...

# Run with coverage
cd server-src && go test ./... -coverprofile=coverage.out && go tool cover -func=coverage.out | tail -n1

# Run with race detector
cd server-src && go test -race ./...

# View coverage report
go tool cover -func=coverage.out
go tool cover -html=coverage.out   # Opens HTML coverage report in browser
```

### Linting and Formatting (app-src directory)
```bash
# JavaScript/JSX linting
cd app-src && yarn lint
cd app-src && yarn lint --fix

# CSS linting
cd app-src && yarn lint:css
cd app-src && yarn lint:css --fix

# Formatting
cd app-src && yarn format
cd app-src && yarn format:check
```

### Configuration Files
- **ESLint:** `app-src/eslint.config.cjs`
- **Stylelint:** `app-src/.stylelintrc.json` (extends stylelint-config-standard, stylelint-config-twbs-bootstrap)
- **Prettier:** `app-src/.prettierrc` (singleQuote, no semi, tabWidth 2, printWidth 80)
- **Jest:** `app-src/jest.config.cjs` (90% coverage threshold)
- **Babel:** `app-src/babel.config.json` (@babel/preset-env, @babel/preset-react)

### Backend Linting (server-src directory)
```bash
# Go linting (using golangci-lint if installed)
golangci-lint run

# Go vet (built-in Go tool)
go vet ./...
```

---

## 3. Code Style Guidelines

### Imports Organization
1. Standard library packages
2. Third-party packages
3. Internal packages

Within each group, sort alphabetically.

### Frontend (JavaScript/React)

#### General Rules
- Use functional components with hooks
- Prefer functional programming patterns
- Use Jotai for state management
- Follow React 19 best practices
- Use PropTypes for component props validation

#### Naming Conventions
- Component names: PascalCase
- Variables/functions: camelCase
- Constants: UPPER_CASE_WITH_UNDERSCORES
- Files: kebab-case (.js, .jsx)
- Test files: *.test.js or *.combined.test.js

#### Imports
- Relative imports for local files
- Absolute imports for external libraries
- Import React at the top of JSX files
- Group imports logically with blank lines between groups

#### Error Handling
- Always handle promise rejections
- Use try/catch with async/await
- Use ErrorBoundary components for UI errors
- Log errors appropriately but avoid exposing sensitive information

### Backend (Go)

#### General Rules
- Follow idiomatic Go patterns
- Use Go modules for dependency management
- Prefer explicit error handling over panics
- Use context.Context for request-scoped values and cancellation

#### Naming Conventions
- Package names: lowercase, single word preferred
- Public functions/types: PascalCase
- Private functions/variables: camelCase with lowercase first letter
- Acronyms: Prefer IDs over Ids, HTTP over Http

#### Error Handling
- Always check errors
- Wrap errors with context where appropriate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heckenmann/docker-swarm-dashboard](https://github.com/heckenmann/docker-swarm-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

---
trigger: always_on
description: Lite Reader is a lightweight RSS feed aggregator written in Go. It allows users to read feeds on their own machine with a simple and minimal application. The application supports multi-user functionality, feed management, and item management.
---

# GitHub Copilot Instructions for Lite Reader

## Project Overview

Lite Reader is a lightweight RSS feed aggregator written in Go. It allows users to read feeds on their own machine with a simple and minimal application. The application supports multi-user functionality, feed management, and item management.

## Technology Stack

- **Language:** Go 1.23.0
- **Database:** SQLite (using modernc.org/sqlite)
- **HTTP Router:** julienschmidt/httprouter
- **Testing:** testify/assert
- **Mocking:** uber/mock (go.uber.org/mock)
- **Build System:** Makefile
- **Dependency Management:** Go modules with vendoring

## Architecture

The project follows a clean architecture pattern:

- `cmd/` - Application entry points
- `internal/app/` - Application business logic layer
  - `auth/` - Authentication and user management
  - `feed/` - Feed management logic
  - `item/` - Item management logic
- `internal/infra/` - Infrastructure layer
  - `http/api/` - HTTP API handlers and middleware
  - `sqlite/` - SQLite repository implementations
  - `job/` - Background job processing
- `internal/config/` - Configuration management
- `internal/mocks/` - Mock implementations for testing
- `public/` - Static assets
- `vendor/` - Vendored dependencies

## Development Workflow

### Running the Application

- `make run` - Run the app without Docker
- `make docker-run` - Run the app with Docker Compose
- `make build` - Build the binary
- `make run-test-server` - Run the app with mock feed server for testing (uses test database)

### Testing

- `make test` - Run all unit tests with coverage
- `make test-ui` - Run UI tests with Playwright (headless)
- `make test-ui-headed` - Run UI tests with visible browser (for debugging)
- `make test-all` - Run both unit and UI tests
- Coverage reports are generated in `reports/` directory
- Aim for high test coverage (current: ~36.3% total, with 100% in feed and item app logic)

#### UI Testing Setup

Before running UI tests for the first time:
- `make test-ui-setup` - Install Playwright and dependencies (one-time setup)

### Code Quality

- `make lint` - Run golangci-lint for code quality checks
- `make gci` - Format imports using gci
- `make pre-commit` - Run all essential checks (gomod, update-mocks, lint, test)
- **Note**: `make pre-commit` runs unit tests but not UI tests. Run `make test-ui` separately before finalizing PRs

### Dependency Management

- `make gomod` - Tidy go.mod and update vendor directory
- Always run `make gomod` after adding or updating dependencies
- Use vendored dependencies (`-mod=vendor` flag)

### Mocks

- `make update-mocks` - Regenerate mocks using mockgen
- Mock definitions are in `internal/mocks/definition.go`
- Generated mocks should not be edited manually

## Coding Standards

### General Guidelines

1. **Follow Go conventions:** Use gofmt, follow effective Go practices
2. **Vendor dependencies:** Always use `-mod=vendor` flag for builds and tests
3. **Test coverage:** Write tests for new features, maintain or improve coverage
4. **Clean architecture:** Keep business logic in `app/` layer, infrastructure in `infra/`
5. **Error handling:** Use proper error wrapping and return descriptive errors
6. **Logging:** Use logrus for structured logging

### File Organization

- Each domain (auth, feed, item) should have:
  - `domain.go` - Domain models and interfaces
  - `service.go` - Service implementation
  - `commands.go` - Command/request structures
  - `dependencies.go` - Dependency injection setup
  - `service_test.go` - Unit tests

### Testing Guidelines

#### Unit Tests
- Use testify for assertions: `assert` package
- Mock external dependencies using uber/mock
- Table-driven tests are preferred for multiple test cases
- Test files should be named `*_test.go`
- Place tests in the same package as the code being tested

#### UI Tests
- UI tests are located in `tests/ui/` directory
- Use Page Object Model pattern (see `tests/ui/pages/`)
- Tests use Playwright with Chromium browser
- **IMPORTANT**: Always use the backend testserver for feed testing instead of fetching feeds from the internet
  - Mock feeds are served from `http://localhost:3001/feeds/`
  - Available mock feeds are defined in `tests/ui/utils/helpers.js` as `MOCK_FEEDS`
  - Never use real external feed URLs in tests to avoid network blockages
- Test database is separate: `data/test-agg.db`
- Tests run sequentially to avoid database conflicts

### Naming Conventions

- Interfaces should be defined where they are used (consumer side)
- Repository interfaces: `*Repository` (e.g., `FeedRepository`)
- Service interfaces: `*Service` (e.g., `FeedService`)
- Commands/Requests: `*Command` (e.g., `CreateFeedCommand`)
- Use descriptive variable names, avoid single-letter names except for common cases (i, j for loops)

### Database Migrations

- Migrations are in `internal/infra/sqlite/migrations/`
- Use goose for migration management
- Migration files should be numbered sequentially

## Common Tasks

### Adding a New Feature

1. Define domain models and interfaces in `internal/app/<domain>/domain.go`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cubny/lite-reader](https://github.com/cubny/lite-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

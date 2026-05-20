---
trigger: always_on
description: Night Routine Scheduler is a Go application that manages night routine scheduling between two parents with Google Calendar integration and optional babysitter assignment support. The application features:
---

# GitHub Copilot Instructions for night-routine

## Project Overview

Night Routine Scheduler is a Go application that manages night routine scheduling between two parents with Google Calendar integration and optional babysitter assignment support. The application features:

- Fair distribution algorithm for parent assignments
- Babysitter assignment support (manual override, treated as shift in fairness calculations)
- Web-based settings UI with real-time updates
- SQLite database for configuration and tracking
- Google OAuth2 authentication and Calendar API integration
- Tailwind CSS v4 for UI styling

## Project Structure

```
night-routine/
├── cmd/night-routine/     # Application entry point
├── internal/              # Internal packages
│   ├── calendar/          # Google Calendar API integration
│   ├── config/            # Configuration management
│   ├── database/          # SQLite database layer
│   ├── fairness/          # Assignment scheduling algorithm
│   │   └── scheduler/     # Core scheduling logic
│   ├── handlers/          # HTTP handlers and web UI
│   │   ├── assets/        # CSS and static assets
│   │   └── templates/     # HTML templates
│   ├── constants/         # Application constants
│   ├── logging/           # Zerolog configuration
│   ├── signals/           # Signal handling
│   ├── token/             # OAuth token management
│   └── viewhelpers/       # Template helper functions
├── docs/                  # Architecture and planning docs
├── docs-site/             # MkDocs documentation site
└── configs/               # Configuration examples
```

## Code Quality Standards

### Formatting

- **Always run `go fix`** on any Go files you modify before committing to apply canonical API migrations
- **Always run `go fmt`** on any Go files you modify before committing
- Use `gofmt -s` for simplified formatting where possible
- Ensure consistent formatting across all Go source files

### Linting

- **Always run `golangci-lint`** to check for code quality issues
- Run `golangci-lint run` to check the entire project
- Run `golangci-lint run ./path/to/package` to check specific packages
- Address all linting issues before committing code
- The project uses a `.golangci.yml` configuration file with custom settings

### Testing

- **Always run tests** before committing: `go test ./...`
- Tests are located alongside source files with `_test.go` suffix
- Use table-driven tests for multiple test cases
- Follow existing test patterns in the codebase
- **Always add a regression unit test** for every bug discovered and fixed to prevent reintroducing the same issue
- Key test areas:
  - `internal/fairness/scheduler/` - Scheduling algorithm tests
  - `internal/handlers/` - HTTP handler tests
  - `internal/database/` - Database operation tests
  - `internal/config/` - Configuration tests

### Building Assets

- **Always run `go generate` before building** to generate CSS and other assets
- Run `go generate ./...` from the project root to generate all assets
- The CSS files are generated using Tailwind CSS v4 via pnpm
- Assets must be regenerated after any template or CSS changes
- The generate directive is in `internal/handlers/base_handler.go`
- Generated CSS is embedded in the binary via `//go:embed` directives

### Build Artifacts and Git

- **Never commit build artifacts** - they are gitignored
- Gitignored items include:
  - Binary: `night-routine` executable
  - Dependencies: `node_modules/`
  - Build output: `dist/`, `bin/`, `_output/`
  - Database files: `data/*.db*`, `test_*.db`
  - Documentation build: `site/`
  - IDE files: `.idea/`, `.vscode/*` (except specific settings)
  - Environment: `.env` file
- Use `.gitignore` to exclude additional temporary or generated files

### Build Process

1. Install Node.js dependencies: `pnpm install --frozen-lockfile`
2. Generate assets: `go generate ./...`
3. Build the application: `go build -o night-routine ./cmd/night-routine`
4. Run tests: `go test ./...`

### Using Go Language Server (gopls)

When working with Go code, **prefer using gopls (Go language server)** for navigation, analysis, and understanding the codebase:

- **Use gopls for code navigation** instead of grep/find when exploring Go code
- **Start gopls in async mode** for complex analysis tasks
- **Leverage gopls capabilities** for:
  - Finding symbol definitions and references
  - Understanding package APIs and dependencies
  - Analyzing cross-file dependencies
  - Searching for symbols across the workspace
  - Getting accurate diagnostics beyond linting

**Example workflows using gopls tools:**

> Note: These examples use gopls tool integration available in the Copilot environment, not direct CLI commands

- Search for a symbol across the workspace: `gopls-go_search` with query: "ScheduleAssignment"
- Get package API summary: `gopls-go_package_api` with packagePaths: ["github.com/belphemur/night-routine/internal/fairness"]
- Find all references to a symbol: `gopls-go_symbol_references` with file: "/path/to/file.go", symbol: "ProcessEvents"
- Get file context and dependencies: `gopls-go_file_context` with file: "/path/to/file.go"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Belphemur/night-routine](https://github.com/Belphemur/night-routine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

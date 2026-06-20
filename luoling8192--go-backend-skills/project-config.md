---
trigger: always_on
description: Initialize a new Go backend repository with production-grade engineering setup. Use when creating new Go microservice projects, setting up Go repos from scratch, or when the user asks to scaffold a Go backend with best practices (Uber FX, Ent ORM, gRPC, CI/CD, linting, Docker).
---


# Go Backend Repository Initialization Skill

Initialize a production-grade Go backend repository following battle-tested patterns.

## When to Use

- User asks to create a new Go backend project
- User wants to scaffold a Go microservice repo
- User needs a Go project template with CI/CD, linting, Docker, etc.

## Initialization Checklist

When invoked, walk through these steps **in order**. Ask the user for project-specific details first:

1. **Project name** (Go module path, e.g., `github.com/org/my-service`)
2. **Service names** (e.g., `platform`, `api`, `worker` — at least one)
3. **Go version** (default: latest stable, currently 1.26)
4. **Database** (PostgreSQL with Ent ORM is default; ask if needed)
5. **Proto/gRPC** (yes/no — if yes, set up buf + grpc-gateway for HTTP/JSON)
6. **System dependencies** (any C libraries like libsoxr, etc.)

## Directory Structure

Create this structure, adapting service names to the user's input:

```
.
├── cmd/
│   └── <service>/
│       └── main.go              # Cobra + Uber FX app entry point
├── internal/
│   ├── configs/                 # Config loading (.env + env vars)
│   │   ├── config.go
│   │   └── module.go
│   ├── datastore/               # Database client initialization (Ent)
│   │   ├── datastore.go
│   │   └── module.go
│   ├── grpc/
│   │   ├── servers/             # gRPC + HTTP server lifecycle
│   │   ├── services/            # gRPC service implementations (business logic)
│   │   └── registers/           # Service registration with gRPC server
│   └── models/                  # Data access layer (Ent ORM CRUD)
├── pkg/                         # Reusable packages
│   ├── apierrors/               # Standardized error handling
│   └── logging/                 # slog + tint colored console logger
├── schema/                      # Ent ORM schema definitions
│   └── generate.go              # go:generate directive for Ent
├── ent/                         # (generated) Ent ORM code — DO NOT EDIT
├── api/
│   ├── proto/<service>/v1/      # .proto source files (if using gRPC)
│   └── generated/               # (generated) Proto/gRPC code — DO NOT EDIT
├── .github/
│   ├── actions/
│   │   └── setup-go-env/
│   │       └── action.yml       # Composite action: Go + system deps
│   └── workflows/
│       └── ci.yml               # Lint + Test + Build + Autofix
├── .vscode/
│   ├── settings.json
│   └── extensions.json
├── .editorconfig
├── .gitignore
├── .golangci.yml
├── .tool-versions
├── .env.example
├── buf.yaml                     # (if using gRPC) Buf CLI config
├── buf.gen.yaml                 # (if using gRPC) Proto code generation
├── cspell.config.yaml
├── renovate.json
├── Makefile                     # build, test, lint, generate, dev
├── Dockerfile
├── docker-compose.yml           # Local dev infra (postgres, etc.)
├── go.mod
└── CLAUDE.md                    # AI assistant context
```

## File Templates

### 1. `go.mod`

```go
module {{MODULE_PATH}}

go {{GO_VERSION}}

require (
	go.uber.org/fx v1.24.0
	// CLI
	github.com/spf13/cobra v1.9.1
	// Logging
	github.com/lmittmann/tint v1.1.3
	// Functional utilities
	github.com/samber/lo v1.50.0
	github.com/nekomeowww/fo v1.3.0
	github.com/nekomeowww/xo v1.9.0
	// Database
	entgo.io/ent v0.14.6
	// gRPC + gateway (if using gRPC)
	google.golang.org/grpc v1.79.3
	google.golang.org/protobuf v1.36.11
	github.com/grpc-ecosystem/grpc-gateway/v2 v2.28.0
	// Testing
	github.com/stretchr/testify v1.11.1
)
```

After creating, run `go mod tidy` to resolve dependencies.

### 2. `.editorconfig`

```ini
root = true

[*.toml]
indent_size = 4
indent_style = space
max_line_length = 100
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[*.{js,ts,vue,tsx,jsx,html,css,json,yaml,yml}]
indent_size = 2
indent_style = space
trim_trailing_whitespace = true

[*.go]
indent_size = 2
indent_style = tab
trim_trailing_whitespace = true

[*.proto]
indent_size = 2
indent_style = space

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
```

### 3. `.golangci.yml`

See [references/golangci.yml](references/golangci.yml) for the full config.

Key principles:
- Use `version: "2"` format
- Start with `default: all`, then disable noisy/opinionated linters
- Enable `gofmt` + `goimports` formatters
- Exclude generated code paths (`api/`, `ent/`) with `generated: lax`
- Set sane thresholds: `dupl: 600`, `nestif: min-complexity: 9`

### 4. `.gitignore`

```gitignore
# Binaries for programs and plugins
*.exe
*.exe~
*.dll
*.so
*.dylib
bin/*
dist/*
out/*

# Test binary, built with `go test -c`
*.test

# Output of the go coverage tool
*.out

# Go workspace file
go.work

# Editor and IDE
.idea
*.swp
*.swo
*~
.DS_Store

# Config files (keep example)
config/*
!config/config.yaml

__debug*
.env
.env.local

# Binary outputs (add your service names)
./main
```

### 5. CI Workflow (`.github/workflows/ci.yml`)

See [references/ci.yml](references/ci.yml) for the full workflow.

Key jobs:
- **Lint** (push only): golangci-lint with 7m timeout
- **Unit Test**: `go test ./...`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luoling8192/go-backend-skills](https://github.com/luoling8192/go-backend-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: Guidance for AI agents working in this codebase.
---

# AGENTS.md - Inbucket

Guidance for AI agents working in this codebase.

## Project Overview

Inbucket is an email testing service that accepts messages for any email address and makes them available via web, REST, and POP3 interfaces. It's a self-contained Go application with an Elm-based web UI.

**Tech Stack:**
- Backend: Go 1.25+
- Frontend: Elm 0.19.1 with Parcel bundler
- Logging: zerolog
- Testing: testify (assert/require/suite), goldiff for golden file tests
- HTTP Router: gorilla/mux
- Configuration: envconfig (environment variables)
- Optional: Lua scripting for extensions (gopher-lua)

## Essential Commands

### Build

```bash
# Build Go binaries (inbucket server + client CLI)
make build

# Or build directly
go build ./cmd/inbucket
go build ./cmd/client

# Build UI (required before running server)
cd ui && yarn install && yarn build
```

### Test

```bash
# Run all Go tests with race detection
make test
# or
go test -race ./...

# Run tests for a specific package
go test -race ./pkg/storage/...

# Run tests with coverage
go test -race -coverprofile=profile.cov ./...
```

### Lint

```bash
# CI uses golangci-lint
golangci-lint run

# Make's lint target (older, uses golint)
make lint
```

### Run Development Server

```bash
# Build everything first
make build
cd ui && yarn build && cd ..

# Run with dev config
./etc/dev-start.sh

# Or run directly with defaults
./inbucket
```

Default ports:
- Web UI: http://localhost:9000
- SMTP: localhost:2500
- POP3: localhost:1100

### UI Development

```bash
cd ui

# Install dependencies
yarn install

# Development server with HMR (proxies to Go backend)
yarn start

# Production build
yarn build

# Clean build artifacts
yarn clean
```

## Code Organization

```
cmd/
  inbucket/           # Main server binary
  client/             # CLI client for REST API

pkg/
  config/             # Environment-based configuration
  extension/          # Lua extension system
    luahost/          # Lua VM pool and bindings
    event/            # Extension event types
  message/            # Message manager (storage abstraction)
  metric/             # Expvar metrics
  msghub/             # Real-time message pub/sub
  policy/             # Email address/domain policies
  rest/               # REST API v1/v2 controllers
    client/           # Go client library for REST API
    model/            # JSON API models
  server/
    smtp/             # SMTP server
    pop3/             # POP3 server
    web/              # HTTP server, handlers, helpers
  storage/            # Storage interface and implementations
    file/             # File-based storage
    mem/              # In-memory storage
  stringutil/         # String utilities
  test/               # Test utilities and integration tests
  webui/              # Web UI controllers

ui/
  src/
    Main.elm          # Elm app entry point
    Api.elm           # API client
    Page/             # Page modules (Home, Mailbox, Monitor, Status)
    Data/             # Data models
  tests/              # Elm tests
```

## Configuration

Inbucket uses environment variables for all configuration. Key variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `INBUCKET_LOGLEVEL` | `info` | debug, info, warn, error |
| `INBUCKET_MAILBOXNAMING` | `local` | local, full, or domain |
| `INBUCKET_SMTP_ADDR` | `0.0.0.0:2500` | SMTP listen address |
| `INBUCKET_WEB_ADDR` | `0.0.0.0:9000` | HTTP listen address |
| `INBUCKET_POP3_ADDR` | `0.0.0.0:1100` | POP3 listen address |
| `INBUCKET_STORAGE_TYPE` | `memory` | `memory` or `file` |
| `INBUCKET_WEB_UIDIR` | `ui/dist` | Path to built UI files |

Run `./inbucket -help` for complete list.

See `doc/config.md` for detailed documentation.

## Code Patterns

### Error Handling
- Use zerolog for structured logging
- Return errors up the call stack; log at the top level
- Use `github.com/pkg/errors` patterns for wrapping

### HTTP Handlers
Handlers follow this pattern in `pkg/server/web/`:
```go
func Handler(f func(http.ResponseWriter, *http.Request, *Context) error) http.Handler
```

Controllers return errors; the wrapper handles HTTP responses.

### Storage Interface
New storage backends implement `storage.Store` interface (`pkg/storage/storage.go`):
```go
type Store interface {
    AddMessage(message Message) (id string, err error)
    GetMessage(mailbox, id string) (Message, error)
    GetMessages(mailbox string) ([]Message, error)
    MarkSeen(mailbox, id string) error
    PurgeMessages(mailbox string) error
    RemoveMessage(mailbox, id string) error
    VisitMailboxes(f func([]Message) (cont bool)) error
}
```

Register in `cmd/inbucket/main.go` init():
```go
storage.Constructors["mytype"] = mystore.New
```

### JSON Tag Convention
JSON fields use kebab-case (configured in `.golangci.yml` tagliatelle):
```go
type Example struct {
    FieldName string `json:"field-name"`
}
```

### Elm Architecture
The UI follows The Elm Architecture:
- `Main.elm` - App shell, routing
- `Page/*.elm` - Page modules with Model, Msg, init, update, view
- `Data/*.elm` - Data types and JSON decoders
- `Api.elm` - HTTP client for REST API

## Testing

### Test Structure
- Unit tests: alongside source files (`*_test.go`)
- Integration tests: `pkg/test/integration_test.go`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inbucket/inbucket](https://github.com/inbucket/inbucket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

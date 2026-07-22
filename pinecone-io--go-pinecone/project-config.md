---
trigger: always_on
description: **Project:** Pinecone Go SDK — Vector database client for AI applications
---

# Pinecone Go SDK - AI Assistant Guide

**Project:** Pinecone Go SDK — Vector database client for AI applications
**Go:** 1.21+
**Module:** `github.com/pinecone-io/go-pinecone/v5`

## Project Overview

The Pinecone Go SDK provides a client for interacting with Pinecone vector databases. Built for type safety, correctness, and idiomatic Go.

**Entry points:**
- `Client` — Control plane client for managing indexes, collections, backups, and inference
- `AdminClient` — Admin operations (projects, orgs, API keys)
- `IndexConnection` — Data plane client for vector operations on a specific index

**Key technologies:**
- Standard `net/http` for REST API calls (Control Plane, Inference, Admin)
- `google.golang.org/grpc` for Data Plane operations
- Auto-generated clients from OpenAPI/Protobuf specs via `oapi-codegen` and `protoc-gen-go`
- `testify` for testing (assert, require, suite)

## Architecture

Three-plane design, each with its own client struct:

**Public API (`pinecone/`):**
- `client.go` — `Client`: Control Plane (create/list/describe/delete indexes, collections, backups) + Inference
- `admin_client.go` — `AdminClient`: Admin operations (projects, orgs, API keys)
- `index_connection.go` — `IndexConnection`: Data Plane over gRPC (upsert, query, fetch, delete, update vectors; namespace management)
- `models.go` — All shared types, constants, and enums (`IndexMetric`, `Cloud`, `IndexStatus`, etc.)
- `errors.go` — `PineconeError` error type

**Internal (`internal/`):**
- `internal/gen/` — Auto-generated client code. **Never edit manually.** Regenerate with `just gen`.
  - `db_control/` — REST client for Control Plane
  - `db_data/grpc/` — gRPC bindings for Data Plane
  - `db_data/rest/` — REST bindings for Data Plane
  - `inference/` — Inference API client
  - `admin/` — Admin API client
- `internal/provider/` — Auth/header provider utilities
- `internal/useragent/` — User-agent string construction
- `internal/utils/` — Internal utility functions

**Code Generation:**
Generated code under `internal/gen/` is produced by `codegen/build-clients.sh` from API specs in `codegen/apis/` (a private git submodule). Run `just gen` to regenerate after spec changes. The current API version is `2025-10`.

## Build & Test Commands

**Setup:**
```bash
# Install required codegen tools (protoc-gen-go, oapi-codegen, godoc)
just bootstrap

# Initialize git submodules (internal Pinecone employees only)
git submodule update --init --recursive
```

**Note:** The `codegen/apis/` submodule contains internal OpenAPI/Protobuf specifications accessible only to Pinecone employees. It is not required for SDK development.

**Build:**
```bash
just build        # go build -v ./... && go vet ./...
just build-clean  # clean build with -a flag
```

**Test:**
```bash
# Unit tests only (fast, no external dependencies)
just test-unit    # go test -v -run Unit ./pinecone

# All tests (requires .env with credentials)
just test         # go test -count=1 -v ./pinecone

# Single test by name
go test -v -run TestNameHere ./pinecone/...
```

**Integration tests require `.env`:**
```
PINECONE_API_KEY=...
PINECONE_CLIENT_ID=...
PINECONE_CLIENT_SECRET=...
```

**Documentation:**
```bash
just docs   # godoc server at http://localhost:6060/pkg/github.com/pinecone-io/go-pinecone/v5/pinecone/
```

**Code generation:**
```bash
just gen    # regenerate internal/gen/ from API specs
```

## Code Style & Conventions

**Quick summary:**

- **Formatting:** Always run `gofmt` (or `go fmt ./...`). All Go code must be gofmt-compliant; non-negotiable.
- **Vetting:** `go vet ./...` before committing. Zero warnings allowed.
- **Naming:** `PascalCase` for exported identifiers; `camelCase` for unexported. Acronyms stay uppercase (`URL`, `HTTP`, `API`, `ID`). Short contextual names in function bodies (`ctx`, `err`, `idx`) are idiomatic.
- **Comments:** All exported types, functions, constants, and variables require GoDoc comments. The comment must start with the name of the symbol (e.g., `// Client holds the parameters...`).
- **Errors:** Return errors rather than panicking. Use `fmt.Errorf("context: %w", err)` to wrap with context. Use `errors.Is()` and `errors.As()` for inspection.
- **Context:** Always accept `context.Context` as the first parameter in functions that perform I/O or long-running operations.
- **Testing:** Unit tests use `*testing.T` with `testify/assert` and `testify/require`. Unit test names must end with `Unit` (e.g., `TestNewClientParamsSetUnit`). Integration tests are methods on the `integrationTests` suite.
- **Generated code:** Never modify `internal/gen/` directly. Regenerate with `just gen`.

**Key patterns:**
- `Client` instances named `pc` by convention in examples and tests
- Option parameters use dedicated request structs (e.g., `CreateServerlessIndexRequest`)
- Public API methods accept `context.Context` as first parameter
- Input validation occurs before API calls; return descriptive `PineconeError` with context

**Module management:**
```bash
go get <package>    # add dependency
go mod tidy         # clean unused dependencies and update go.sum
```

## Security Considerations

**Always:**
- Redact API keys in logs and error messages — never include raw credentials in error output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pinecone-io/go-pinecone](https://github.com/pinecone-io/go-pinecone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

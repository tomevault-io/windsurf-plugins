---
trigger: always_on
description: > RAGent: CLI tool for building RAG systems from Markdown documents using hybrid search (BM25 + vector) with Amazon S3 Vectors and OpenSearch.
---

# AGENTS.md — ragent

> RAGent: CLI tool for building RAG systems from Markdown documents using hybrid search (BM25 + vector) with Amazon S3 Vectors and OpenSearch.

## Build / Lint / Test Commands

```bash
# Build
go build -o ragent .

# Lint (full: fmt + vet + golangci-lint)
make lint

# Lint (CI subset: fmt + vet only)
make fmtvet

# Unit tests (excludes tests/ directory which contains integration/contract/benchmark)
go test $(go list ./... | grep -v '/tests/') -timeout 120s

# Single test function
go test -v -run TestFunctionName ./path/to/package/ -timeout 30s

# Single test with subtest
go test -v -run "TestParent/subtest_name" ./path/to/package/

# Unit tests in tests/unit/
go test -v ./tests/unit/... -timeout 60s

# Contract tests
go test -v ./tests/contract/... -timeout 60s

# Integration / E2E tests (requires OpenSearch via docker compose)
make test                # Full E2E: starts OpenSearch, creates index, runs tests
make test-teardown       # Cleanup after E2E

# E2E tests directly (OpenSearch must be running)
OPENSEARCH_ENDPOINT=http://localhost:9200 OPENSEARCH_INDEX=ragent-e2e-test \
  go test -v -run "TestE2E_" ./tests/integration/... -timeout 120s

# Benchmarks
go test -bench=. ./tests/benchmark/... -timeout 120s
```

## Architecture — Vertical Slice

This codebase follows a **vertical slice** architecture. Each feature slice lives under `internal/` and owns its full stack: types, interfaces, handlers, and tests.

```
main.go                        # Entrypoint → cmd.Execute()
cmd/                           # Cobra command definitions (thin wiring layer)
  root.go                      # Root command, subcommand registration
  vectorize.go, query.go, ...  # Each delegates to internal/<slice>
internal/
  ingestion/                   # Vectorization pipeline slice
    command.go                 #   Orchestrator (called from cmd/vectorize.go)
    vectorizer/                #   Core service, interfaces, error handling
    scanner/                   #   File scanning (local, S3, GitHub)
    metadata/                  #   Metadata extraction
    csv/, spreadsheet/, pdf/   #   Format-specific readers
    hashstore/                 #   Change detection for incremental processing
  mcpserver/                   # MCP server slice
    command.go                 #   Server bootstrap (called from cmd/mcp-server.go)
    server.go, server_wrapper.go
    hybrid_search_handler.go   #   Tool handler
    auth_middleware.go, oidc_auth.go, ip_auth.go
    types.go                   #   All MCP-specific types
  query/                       # Query/Chat slice
  slackbot/                    # Slack bot slice
  webui/                       # Web dashboard slice
  pkg/                         # Shared packages (cross-slice)
    config/                    #   Central config (env vars via netflix/go-env)
    domain/                    #   Shared interfaces & types
    embedding/                 #   Embedding clients (Bedrock, Gemini)
    opensearch/                #   OpenSearch client & hybrid engine
    s3vector/, sqlitevec/      #   Vector store backends
    slacksearch/               #   Slack search service
    metrics/, observability/   #   Telemetry
    ipc/                       #   Inter-process communication
    evalexport/                #   Eval export
tests/
  unit/                        # Unit tests (no external deps)
  contract/                    # Protocol/interface contract tests
  integration/                 # E2E tests (require OpenSearch, AWS)
  benchmark/, benchmarks/      # Performance benchmarks
  testdata/                    # Test fixtures (index mappings, etc.)
```

### Vertical Slice Rules

1. **Each slice owns its types.** Define domain types in the slice's `types.go`, not in `pkg/domain/`.
2. **Shared types go in `internal/pkg/domain/`** only when genuinely used across 2+ slices.
3. **`cmd/` is thin wiring.** Parse flags, build options struct, call `internal/<slice>.RunXxx(cmd, opts)`. No business logic.
4. **Inter-slice communication** via interfaces in `internal/pkg/domain/interfaces.go`.
5. **New features** → new directory under `internal/` or extend an existing slice. Never put business logic in `cmd/`.

## Code Style

### Imports (enforced by goimports)

Three groups separated by blank lines, ordered: stdlib → third-party → internal.

```go
import (
    "context"
    "fmt"

    "github.com/spf13/cobra"
    "github.com/stretchr/testify/assert"

    appconfig "github.com/ca-srg/ragent/internal/pkg/config"
    "github.com/ca-srg/ragent/internal/pkg/domain"
)
```

Local prefix: `github.com/ca-srg/ragent` (configured in `.golangci-lint.yml`).

### Formatting & Line Length

- `gofmt` and `goimports` — enforced.
- Max line length: **140 characters** (enforced by `lll` linter).
- Tab width: 4 for line length calculation.

### Naming Conventions

- **Packages**: lowercase, single-word (`vectorizer`, `scanner`, `config`). No underscores.
- **Interfaces**: descriptive verbs/nouns (`EmbeddingClient`, `VectorStore`, `FileScanner`). No `I` prefix.
- **Structs**: PascalCase (`VectorizerService`, `SearchError`, `HybridSearchRequest`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ca-srg/ragent](https://github.com/ca-srg/ragent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->

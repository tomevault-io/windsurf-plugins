---
trigger: always_on
description: This document provides essential context for AI agents working on the `dbn-go` project.
---

# AGENTS.md - Agent Guidelines for dbn-go

This document provides essential context for AI agents working on the `dbn-go` project.

## Project Overview

**dbn-go** is a Go library and CLI toolkit for Databento's DBN (Databento Binary Encoding) format and APIs. It is **not affiliated with Databento** - it's an independent open-source project by Neomantra Corp / Nimble Markets.

Key capabilities:
- DBN file reading/writing (binary and JSON formats)
- Historical API client (`hist/`)
- Live API client (`live/`)
- 6 CLI tools for file processing, API interaction, TUI, docs scraping, and MCP server

## Project Structure

```
.
├── *.go                    # Core library: structs, consts, scanners, metadata
├── hist/                   # Historical API client
├── live/                   # Live API client  
├── cmd/                    # CLI tools
│   ├── dbn-go-file/        # DBN file processing (parquet, split, etc.)
│   ├── dbn-go-hist/        # Historical API CLI
│   ├── dbn-go-live/        # Live feed handler
│   ├── dbn-go-mcp/         # MCP (Model Context Protocol) server
│   ├── dbn-go-slurp-docs/  # Databento docs scraper
│   └── dbn-go-tui/         # Terminal UI for Databento account
├── internal/               # Internal packages
│   ├── file/               # File processing (JSON writer, Parquet, split)
│   └── tui/                # TUI components
├── tests/                  # Test data and shell scripts
└── tests/data/             # Sample DBN files
```

## Build System

Use **Task** (taskfile.yml) for builds:

```bash
task              # Default: test and build
task build        # Build all binaries to bin/
task test         # Run Go tests
task test-all     # Run tests + CLI tool tests
task clean        # Remove built binaries
task list         # Show all tasks
```

Direct Go commands also work:
```bash
go build ./...
go test ./...
```

## Testing

- **Framework**: Ginkgo v2 + Gomega (BDD style)
- **Test package**: `dbn_test` (external) for most tests
- **Test data**: Sample DBN files in `tests/data/`
- **API tests**: Some tests require `DATABENTO_API_KEY` env var

Running tests without API calls:
```bash
go test && go test ./live
```

## Code Style

### File Header
All Go files must include:
```go
// Copyright (c) 2024-2025 Neomantra Corp
```

### Naming Conventions
- **Exported types**: PascalCase (e.g., `DbnScanner`, `OhlcvMsg`)
- **Enums**: SCREAMING_SNAKE_CASE for constants with type prefix
  - e.g., `Schema_Ohlcv1S`, `SType_InstrumentId`, `RType_Mbp0`
- **Private helpers**: camelCase with `_Raw` suffix for binary decoders
- **Generic constraints**: `RecordPtr[T]` interface for type-safe reading

### Package Organization
- Root package: `dbn`
- Historical API: `dbn_hist` (import path `github.com/NimbleMarkets/dbn-go/hist`)
- Live API: `dbn_live` (import path `github.com/NimbleMarkets/dbn-go/live`)

### Error Handling
- Use sentinel errors defined in `errors.go`
- Wrap errors with context: `fmt.Errorf("context: %w", err)`
- Explicit error returns, no panic for expected errors

## Key Patterns

### 1. Scanner Pattern (Streaming)
```go
scanner := dbn.NewDbnScanner(reader)
for scanner.Next() {
    record, err := dbn.DbnScannerDecode[dbn.OhlcvMsg](scanner)
    // handle record
}
```

### 2. Visitor Pattern (Type Dispatch)
```go
type MyVisitor struct{}
func (v *MyVisitor) OnOhlcv(record *dbn.OhlcvMsg) error { ... }
// ... implement other OnXxx methods

scanner.Visit(visitor)  // Dispatches to appropriate handler
```

### 3. Generic Slice Reading
```go
records, metadata, err := dbn.ReadDBNToSlice[dbn.OhlcvMsg](reader)
```

### 4. Compression Handling
```go
file, closer, err := dbn.MakeCompressedReader("file.dbn.zstd", false)
defer closer.Close()
```

## MCP Server (`cmd/dbn-go-mcp`)

The MCP server (`dbn-go-mcp`) bridges LLMs and Databento's Historical API via the Model Context Protocol. Key architecture notes:

- **Single file**: All code is in `cmd/dbn-go-mcp/main.go`
- **Library**: Uses `github.com/mark3labs/mcp-go` for MCP protocol handling
- **Transports**: STDIO (default) and SSE
- **Error convention**: Tool handlers return errors as `mcp.NewToolResultError()` (not Go errors), so the LLM can see and reason about failures
- **Shared validation**: `parseCommonParams()` extracts/validates dataset, schema, symbol, start, end from tool requests; `commonParams.metadataQueryParams()` builds the hist API query struct
- **Cost guard**: `get_range` checks estimated cost against `--max-cost` budget before fetching data

### MCP Tools (10 total)

Discovery tools (no billing): `list_datasets`, `list_publishers`, `list_schemas`, `list_fields`, `get_dataset_range`, `get_dataset_condition`, `list_unit_prices`, `resolve_symbols`

Query tools: `get_cost` (metadata only), `get_range` (incurs billing, returns JSON)

### Adding New Tools

1. Define the tool with `mcp.NewTool()` in `registerTools()`, using `mcp.WithDescription()` and typed parameters
2. Write a handler `func(ctx, mcp.CallToolRequest) (*mcp.CallToolResult, error)` — return errors via `mcp.NewToolResultErrorf()`, not as Go errors
3. For tools using dataset/schema/symbol/date params, reuse `parseCommonParams()`
4. Register with `mcpServer.AddTool(tool, handler)`

## DBN Format Notes

- **Encoding**: Little-endian binary

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NimbleMarkets/dbn-go](https://github.com/NimbleMarkets/dbn-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

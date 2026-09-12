---
trigger: always_on
description: FinFocus is a CLI tool for calculating cloud infrastructure costs from Pulumi infrastructure definitions. It uses a plugin-based architecture to query multiple cost data sources via gRPC.
---

# FinFocus Development Instructions

FinFocus is a CLI tool for calculating cloud infrastructure costs from Pulumi infrastructure definitions. It uses a plugin-based architecture to query multiple cost data sources via gRPC.

## Build, Test, and Lint Commands

```bash
make build                              # Build binary to bin/finfocus
make build-all                          # Build binary + all plugins (recorder, Pulumi tool)
make clean                              # Remove build artifacts

make test                               # Run unit tests (fast, default)
go test ./internal/cli/...              # Test specific package
go test -run TestName ./...             # Run single test by name
go test -v -run TestName ./internal/engine/...  # Single test in one package

make test-race                          # Run with race detector
make test-integration                   # Integration tests (slower)
make test-e2e                           # E2E tests (requires AWS credentials)

make lint                               # golangci-lint v2.9.0 + markdownlint + actionlint
make validate                           # go mod tidy -diff + go vet
make docs-lint                          # Lint documentation only

go test -coverprofile=coverage.out ./...  # Coverage report
go tool cover -html=coverage.out
```

**Always run `make lint` and `make test` before committing.**

## Architecture

```text
Pulumi JSON → Ingestion → Resource Descriptors → Router → Engine → Plugins (gRPC) / Specs (YAML) → Output
```

### Dual-Mode Binary

The `finfocus` binary runs as both a standalone CLI and a Pulumi tool plugin. Mode is detected via binary name (`pulumi-tool-finfocus`) or `FINFOCUS_PLUGIN_MODE=true`. This changes the `Use` string and examples but shares all subcommands.

### Core Components

1. **CLI** (`internal/cli/`) — Cobra commands: `overview`, `cost projected|actual|recommendations|budget|estimate`, `plugin *`, `config *`, `analyzer serve`
2. **Engine** (`internal/engine/`) — Orchestrates cost calculation. Tries plugins first, falls back to local YAML specs in `specs/`. Uses `hoursPerMonth = 730`. Supports table, JSON, NDJSON output. Includes batch processing (threshold: 100 resources), caching, and budget forecasting.
3. **Router** (`internal/router/`) — Routes resource types to the correct plugin based on provider patterns, priority rules, and config-driven routing from `~/.finfocus/config.yaml`.
4. **Proto Adapter** (`internal/proto/`) — Bridge between engine and plugins. Converts `ResourceDescriptor` to protobuf requests, performs pre-flight validation via `pluginsdk`, extracts SKU/Region from resource `Inputs`, and aggregates errors with `CostResultWithErrors`.
5. **Plugin Host** (`internal/pluginhost/`) — gRPC plugin lifecycle. `ProcessLauncher` (TCP) and `StdioLauncher` (stdin/stdout). 10-second timeout, 100ms retry. **Always call `cmd.Wait()` after `Kill()` to prevent zombies.**
6. **Registry** (`internal/registry/`) — Discovers plugins at `~/.finfocus/plugins/<name>/<version>/`. Optional `plugin.manifest.json` validation.
7. **Ingestion** (`internal/ingest/`) — Parses `pulumi preview --json`. **Must inspect `newState` to extract `Inputs`** — without this, property extraction fails and plugins return `InvalidArgument`.
8. **Analyzer** (`internal/analyzer/`) — Implements `pulumirpc.AnalyzerServer` for zero-click cost estimation during `pulumi preview`. Prints ONLY port number to stdout (Pulumi handshake). All logs go to stderr. ADVISORY enforcement only.
9. **Config** (`internal/config/`) — Manages `~/.finfocus/config.yaml` including plugin routing rules, budget definitions, and dismissed recommendation state (`~/.finfocus/dismissed.json`).
10. **TUI** (`internal/tui/`) — Bubble Tea + Lip Gloss with adaptive color schemes.

### Plugin Communication

Plugins communicate via gRPC using protocol buffers from [finfocus-spec](https://github.com/rshade/finfocus-spec). Always use `pluginsdk` constants for environment variables and metadata keys:

```go
pluginsdk.EnvPort              // "FINFOCUS_PLUGIN_PORT"
pluginsdk.EnvLogLevel          // "FINFOCUS_LOG_LEVEL"
pluginsdk.EnvLogFormat         // "FINFOCUS_LOG_FORMAT"
pluginsdk.TraceIDMetadataKey   // "x-finfocus-trace-id" (gRPC metadata)
```

Trace IDs propagate automatically: `TraceInterceptor()` in pluginhost injects them into outgoing gRPC calls; plugins extract via `pluginsdk.TracingUnaryServerInterceptor()`.

### Pre-Flight Validation Pattern

The proto adapter validates requests before making gRPC calls. Invalid resources get a `$0` placeholder with `"VALIDATION: ..."` notes (distinct from plugin errors prefixed `"ERROR:"`):

```go
if err := pluginsdk.ValidateProjectedCostRequest(protoReq); err != nil {
    log.Warn().Str("resource_type", resource.Type).Err(err).Msg("pre-flight validation failed")
    result.Results = append(result.Results, &CostResult{
        Currency: "USD", MonthlyCost: 0,
        Notes: fmt.Sprintf("VALIDATION: %v", err),
    })
    continue
}
```

### Recorder Plugin


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rshade/finfocus](https://github.com/rshade/finfocus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->

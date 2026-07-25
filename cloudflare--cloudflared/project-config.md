---
trigger: always_on
description: Cloudflare's command-line tool and networking daemon written in Go.
---

# Cloudflared

Cloudflare's command-line tool and networking daemon written in Go.
Production-grade tunneling and network connectivity services used by millions of
developers and organizations worldwide.

## Essential Commands

### Build & Test (Always run before commits)

```bash
# Full development check (run before any commit)
make test lint

# Build for current platform
make cloudflared

# Run all unit tests with coverage
make test
make cover

# Run specific test
go test -run TestFunctionName ./path/to/package

# Run tests with race detection
go test -race ./...
```

### Platform-Specific Builds

```bash
# Linux
TARGET_OS=linux TARGET_ARCH=amd64 make cloudflared

# Windows
TARGET_OS=windows TARGET_ARCH=amd64 make cloudflared

# macOS ARM64
TARGET_OS=darwin TARGET_ARCH=arm64 make cloudflared

# FIPS compliant build
FIPS=true make cloudflared
```

### Code Quality & Formatting

```bash
# Run linter (38+ enabled linters)
make lint

# Auto-fix formatting
make fmt
gofmt -w .
goimports -w .

# Security scanning
make vet

# Component tests (Python integration tests)
cd component-tests && python -m pytest test_file.py::test_function_name
```

Notes on linting:

- `.golangci.yaml` is configured with `new-from-rev` and `whole-files: true`.
  Touching a file triggers linting of the ENTIRE file, not just the changed
  hunks. Expect to fix pre-existing issues in files you modify, or add
  targeted `// nolint: <linter>` comments with a short justification.
- Prefer `defer func() { _ = resource.Close() }()` over `defer resource.Close()`
  for `io.Closer` values whose error truly does not matter — this satisfies
  `errcheck` without hiding real failures elsewhere.

## Project Knowledge

### Package Structure

- Use meaningful package names that reflect functionality
- Package names should be lowercase, single words when possible
- Avoid generic names like `util`, `common`, `helper`

#### Well-known shared packages

- `crypto/`: Single source of truth for TLS curve preferences and other
  cryptographic primitives shared by every edge-facing transport. Import as
  `cfdcrypto "github.com/cloudflare/cloudflared/crypto"` to avoid colliding
  with the standard library's `crypto` package. Do NOT duplicate TLS curve
  or cipher selection logic in other packages.
- `tlsconfig/`: Builds the base `*tls.Config` used for edge connections
  (`CreateTunnelConfig`) and loads origin/CA pools. Curve selection is
  intentionally NOT set here; it is applied per-connection from the
  `crypto/` package so the same config can be cloned and reused across
  protocols.
- `features/`: Runtime feature flags including `PostQuantumMode`
  (`PostQuantumPrefer` = default, `PostQuantumStrict` = `--post-quantum`).
- `fips/`: Build-tag driven FIPS detection. Only `fips.IsFipsEnabled()` is
  exposed; never branch on `fipsEnabled` inside a function if the two
  branches return the same value.

### Function and Method Guidelines

```go
// Good: Clear purpose, proper error handling
func (c *Connection) HandleRequest(ctx context.Context, req *http.Request) error {
    if req == nil {
        return errors.New("request cannot be nil")
    }
    // Implementation...
    return nil
}
```

### Error Handling

- Always handle errors explicitly, never ignore them
- Use `fmt.Errorf` for error wrapping
- Create meaningful error messages with context
- Use error variables for common errors

```go
// Good error handling patterns
if err != nil {
    return fmt.Errorf("failed to process connection: %w", err)
}
```

### Logging Standards

- Use `github.com/rs/zerolog` for structured logging
- Include relevant context fields
- Use appropriate log levels (Debug, Info, Warn, Error)

```go
logger.Info().
    Str("tunnelID", tunnel.ID).
    Int("connIndex", connIndex).
    Msg("Connection established")
```

### Testing Patterns

- Use `github.com/stretchr/testify` for assertions
- Test files end with `_test.go`
- Use table-driven tests for multiple scenarios
- Always use `t.Parallel()` for parallel-safe tests
- Use meaningful test names that describe behavior

```go
func TestMetricsListenerCreation(t *testing.T) {
    t.Parallel()
    // Test implementation
    assert.Equal(t, expected, actual)
    require.NoError(t, err)
}
```

### Constants and Variables

```go
const (
    MaxGracePeriod       = time.Minute * 3
    MaxConcurrentStreams = math.MaxUint32
    LogFieldConnIndex    = "connIndex"
)

var (
    // Group related variables
    switchingProtocolText = fmt.Sprintf("%d %s", http.StatusSwitchingProtocols, http.StatusText(http.StatusSwitchingProtocols))
    flushableContentTypes = []string{sseContentType, grpcContentType, sseJsonContentType}
)
```

### Type Definitions

- Define interfaces close to their usage
- Keep interfaces small and focused
- Use descriptive names for complex types

```go
type TunnelConnection interface {
    Serve(ctx context.Context) error
}

type TunnelProperties struct {
    Credentials    Credentials
    QuickTunnelUrl string
}
```

## Key Architectural Patterns

### Context Usage

- Always accept `context.Context` as first parameter for long-running operations
- Respect context cancellation in loops and blocking operations
- Pass context through call chains

### Concurrency

- Use channels for goroutine communication

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/cloudflared](https://github.com/cloudflare/cloudflared) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

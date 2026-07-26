---
trigger: always_on
description: **Every site on HTTPS.** Caddy is a security-first, modular, extensible server platform.
---

# Caddy Project Guidelines

## Mission

**Every site on HTTPS.** Caddy is a security-first, modular, extensible server platform.

## Code Style

### Go Idioms

Follow [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments):

- **Error flow**: Early return, indent error handling—not else blocks
  ```go
  if err != nil {
      return err
  }
  // normal code
  ```
- **Naming**: initialisms (`URL`, `HTTP`, `ID`—not `Url`, `Http`, `Id`)
- **Receiver names**: 1–2 letters reflecting type (`c` for `Client`, `h` for `Handler`)
- **Error strings**: Lowercase, no trailing punctuation (`"something failed"` not `"Something failed."`)
- **Doc comments**: Full sentences starting with the name being documented
  ```go
  // Handler serves HTTP requests for the file server.
  type Handler struct { ... }
  ```
- **Empty slices**: `var t []string` (nil slice), not `t := []string{}` (non-nil zero-length)
- **Don't panic**: Use error returns for normal error handling

### Caddy Patterns

**Module registration**:
```go
func init() {
    caddy.RegisterModule(MyModule{})
}

func (MyModule) CaddyModule() caddy.ModuleInfo {
    return caddy.ModuleInfo{
        ID:  "namespace.category.name",
        New: func() caddy.Module { return new(MyModule) },
    }
}
```

**Module lifecycle**: `New()` → JSON unmarshal → `Provision()` → `Validate()` → use → `Cleanup()`

**Interface guards** — compile-time verification that modules implement required interfaces:
```go
var (
    _ caddy.Provisioner     = (*MyModule)(nil)
    _ caddy.Validator       = (*MyModule)(nil)
    _ caddyfile.Unmarshaler = (*MyModule)(nil)
)
```

**Structured logging** — use the module-scoped logger from context:
```go
func (m *MyModule) Provision(ctx caddy.Context) error {
    m.logger = ctx.Logger()
    m.logger.Debug("provisioning", zap.String("field", m.Field))
    return nil
}
```

**Caddyfile support** — implement `UnmarshalCaddyfile(*caddyfile.Dispenser)` using the `Dispenser` API:
```go
// UnmarshalCaddyfile sets up the module from Caddyfile tokens. Syntax:
//
//     directive [arg1] [arg2] {
//         subdir value
//     }
func (m *MyModule) UnmarshalCaddyfile(d *caddyfile.Dispenser) error {
    d.Next() // consume directive name
    for d.NextArg() {
        // handle inline arguments
    }
    for nesting := d.Nesting(); d.NextBlock(nesting); {
        switch d.Val() {
        case "subdir":
            if !d.NextArg() {
                return d.ArgErr()
            }
            m.Field = d.Val()
        default:
            return d.Errf("unrecognized subdirective: %s", d.Val())
        }
    }
    return nil
}
```

**Admin API**: Implement `caddy.AdminRouter` for custom endpoints.

**Context**: Use `caddy.Context` for accessing other apps/modules and logging—don't store contexts in structs.

## Architecture

Caddy is built around a **module system** where everything is a module registered via `caddy.RegisterModule()`:

- **Apps** (`caddy.App`): Top-level modules like `http`, `tls`, `pki` that Caddy loads and runs
- **Modules** (`caddy.Module`): Extensible components with namespaced IDs (e.g., `http.handlers.file_server`)
- **Configuration**: Native JSON with adapters (Caddyfile → JSON via `caddyconfig/httpcaddyfile`)

| Directory | Purpose |
|-----------|---------|
| `modules/` | All standard modules (HTTP, TLS, PKI, etc.) |
| `modules/standard/imports.go` | Standard module registry |
| `caddyconfig/httpcaddyfile/` | Caddyfile → JSON adapter for HTTP |
| `caddytest/` | Test utilities and integration tests |
| `cmd/caddy/` | CLI entry point with module imports |

### Critical Packages

`caddyhttp` and `caddytls` require **extra scrutiny** in code review—these are security-critical.

Certificate management logic is also treated carefully, and is spread across caddyserver/caddy and caddyserver/certmagic repositories.

## Quality Gates


**All required before PR is merge-ready:**

| Gate | Command | Notes |
|------|---------|-------|
| Tests pass | `go test -race -short ./...` | Race detection enabled |
| Lint clean | `golangci-lint run --timeout 10m` | No warnings in changed files |
| Builds | `go build ./...` | Must compile |
| Benchmarks | `go test -bench=. -benchmem` | Required for optimizations |

CI runs tests on **Linux, macOS, and Windows**—ensure cross-platform compatibility.

### Build & Test

```bash
# Build
cd cmd/caddy && go build

# Tests with race detection (matches CI)
go test -race -short ./...

# Integration tests
go test ./caddytest/integration/...

# Lint (matches CI)
golangci-lint run --timeout 10m
```

## Testing Conventions

**Table-driven tests** (preferred pattern):
```go
func TestFeature(t *testing.T) {
    for i, tc := range []struct {
        input    string
        expected string
        wantErr  bool
    }{
        {input: "valid", expected: "result", wantErr: false},
        {input: "invalid", expected: "", wantErr: true},
    } {
        actual, err := Function(tc.input)
        if tc.wantErr && err == nil {
            t.Errorf("Test %d: expected error but got none", i)
        }
        if !tc.wantErr && err != nil {
            t.Errorf("Test %d: unexpected error: %v", i, err)
        }
        if actual != tc.expected {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caddyserver/caddy](https://github.com/caddyserver/caddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

---
trigger: always_on
description: This document provides guidelines for GitHub Copilot when assisting with development in the go-zero project.
---

# GitHub Copilot Instructions for go-zero

This document provides guidelines for GitHub Copilot when assisting with development in the go-zero project.

## Project Overview

go-zero is a web and RPC framework with lots of built-in engineering practices designed to ensure the stability of busy services with resilience design. It has been serving sites with tens of millions of users for years.

### Key Architecture Components

- **REST API framework** (`rest/`) - HTTP service framework with middleware chain support
- **RPC framework** (`zrpc/`) - gRPC-based RPC framework with etcd service discovery and p2c_ewma load balancing
- **Gateway** (`gateway/`) - API gateway supporting both HTTP and gRPC upstreams with proto-based routing
- **MCP Server** (`mcp/`) - Model Context Protocol server for AI agent integration via SSE
- **Core utilities** (`core/`) - Production-grade components:
  - Resilience: circuit breakers (`breaker/`), rate limiters (`limit/`), adaptive load shedding (`load/`)
  - Storage: SQL with cache (`stores/sqlc/`), Redis (`stores/redis/`), MongoDB (`stores/mongo/`)
  - Concurrency: MapReduce (`mr/`), worker pools (`executors/`), sync primitives (`syncx/`)
  - Observability: metrics (`metric/`), tracing (`trace/`), structured logging (`logx/`)
- **Code generation tool** (`tools/goctl/`) - CLI tool for generating Go code from `.api` and `.proto` files

## Coding Standards and Conventions

### Code Style

1. **Follow Go conventions**: Use `gofmt` for formatting, follow effective Go practices
2. **Package naming**: Use lowercase, single-word package names when possible
3. **Error handling**: Always handle errors explicitly, use `errorx.BatchError` for multiple errors
4. **Context propagation**: Always pass `context.Context` as the first parameter for functions that may block
5. **Configuration structures**: Use struct tags with JSON annotations, defaults, and validation

**Pattern**: All service configs embed `service.ServiceConf` for common fields (Name, Log, Mode, Telemetry)
```go
type Config struct {
    service.ServiceConf              // Always embed for services
    Host     string `json:",default=0.0.0.0"`
    Port     int    // Required field (no default)
    Timeout  int64  `json:",default=3000"`  // Timeouts in milliseconds
    Optional string `json:",optional"`      // Optional field
    Mode     string `json:",default=pro,options=dev|test|rt|pre|pro"`  // Validated options
}
```

**Service modes**: `dev`/`test`/`rt` disable load shedding and stats; `pre`/`pro` enable all resilience features

### Interface Design

1. **Small interfaces**: Follow Go's preference for small, focused interfaces
2. **Context methods**: Provide both context and non-context versions of methods
3. **Options pattern**: Use functional options for complex configuration

Example:
```go
func (c *Client) Get(key string, val any) error {
    return c.GetCtx(context.Background(), key, val)
}

func (c *Client) GetCtx(ctx context.Context, key string, val any) error {
    // implementation
}
```

### Testing Patterns

1. **Test file naming**: Use `*_test.go` suffix
2. **Test function naming**: Use `TestFunctionName` pattern
3. **Use testify/assert**: Prefer `assert` package for assertions
4. **Table-driven tests**: Use table-driven tests for multiple scenarios
5. **Mock interfaces**: Use `go.uber.org/mock` for mocking
6. **Test helpers**: Use `redistest`, `mongtest` helpers for database testing

Example test pattern:
```go
func TestSomething(t *testing.T) {
    tests := []struct {
        name     string
        input    string
        expected string
        wantErr  bool
    }{
        {"valid case", "input", "output", false},
        {"error case", "bad", "", true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := SomeFunction(tt.input)
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)
            assert.Equal(t, tt.expected, result)
        })
    }
}
```

## Framework-Specific Guidelines

### REST API Development

1. **API Definition**: Use `.api` files to define REST APIs with goctl codegen
2. **Handler pattern**: Separate business logic into logic packages (handlers call logic layer)
3. **Middleware chain**: Middlewares wrap via `chain.Chain` interface - use `Append()` or `Prepend()` to control order
   - Built-in middlewares (all in `rest/handler/`): tracing, logging, metrics, recovery, breaker, shedding, timeout, maxconns, maxbytes, gunzip
   - Custom middleware: `func(http.Handler) http.Handler` - call `next.ServeHTTP(w, r)` to continue chain
4. **Response handling**: Use `httpx.WriteJson(w, code, v)` for JSON responses
5. **Error handling**: Use `httpx.Error(w, err)` or `httpx.ErrorCtx(ctx, w, err)` for HTTP error responses
6. **Route registration**: Routes defined with `Method`, `Path`, and `Handler` - wildcards use `:param` syntax

### RPC Development

1. **Protocol Buffers**: Use protobuf for service definitions, generate code with goctl
2. **Service discovery**: Use etcd for dynamic service registration/discovery, or direct endpoints for static routing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeromicro/go-zero](https://github.com/zeromicro/go-zero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

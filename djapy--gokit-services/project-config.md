---
trigger: always_on
description: Manages the lifecycle of multiple services. Shutdown is triggered by: SIGINT/SIGTERM, ctx cancellation, a service error, or `ep.Shutdown()`.
---

# gokit-services

Reusable Go toolkit for microservices. Go 1.26. Module: `github.com/DjaPy/gokit-services`.

## Structure

All library code lives under `pkg/`; `example/` and `docs/` sit alongside it at the repo root.

```
pkg/
  core/
    entrypoint/   — application lifecycle
    service/      — Service, Shutdown, Prober interfaces
  http/
    server/       — HTTP server with Prometheus and panic recovery (package server)
    client/       — HTTP client with a middleware chain (package client)
  grpc/
    server/       — gRPC server (package server)
    client/       — gRPC client (package client)
  kafka/          — dialer/TLS/SASL + producer/ and consumer/ subpackages
  healthserver/ periodic/ workerpool/ dbservice/ redisservice/  — infra services
  internal/       — shared helpers (retry, prom), not importable by consumers
example/          — runnable orders-service wiring every primitive
docs/             — SPDD analysis/prompt artifacts
```

Transports are grouped by protocol, and the contract/orchestration layer is moved
into `pkg/core/`. The `server`/`client` subpackages are best imported with aliases
(`httpsrv`, `httpcli`, `grpcsrv`, `grpccli`) — this avoids collisions of the generic
names and the clash with stdlib `net/http`. The infrastructure services (`healthserver`,
`periodic`, `workerpool`, `dbservice`, `redisservice`) remain leaf packages under `pkg/`.

## Interfaces (pkg/core/service)

```go
type Service interface {
    Start(ctx context.Context) error  // blocks until stopped; ctx is canceled on shutdown
}

type Shutdown interface {
    Stop(ctx context.Context) error  // optional graceful stop
}
```

## entrypoint

Manages the lifecycle of multiple services. Shutdown is triggered by: SIGINT/SIGTERM, ctx cancellation, a service error, or `ep.Shutdown()`.

Order: PreStart hooks → Start (concurrently) → PostStart hooks → wait → PreStop hooks → Stop (concurrently) → PostStop hooks.

```go
import "github.com/DjaPy/gokit-services/pkg/core/entrypoint"

ep := entrypoint.New(
    entrypoint.WithServices(httpSrv, grpcSrv),
    entrypoint.WithShutdownTimeout(30 * time.Second),
    entrypoint.WithPreStart(func(ctx context.Context) error { ... }),
)
ep.Run(ctx)
```

## http/server

The HTTP server implements `service.Service` and `service.Shutdown`. It automatically collects Prometheus metrics and recovers from panics.

**Metrics:** `http_request_duration_seconds`, `http_response_size_bytes`, `http_requests_inflight`, `http_panic_recovery_total`. Labels: `http_service`, `http_handler`, `http_method`, `http_code`.

**Important:** Always pass `WithPrometheusRegisterer(prometheus.NewRegistry())` in tests — otherwise a second `NewServer` with the default registerer panics on duplicate metric registration. In production, use a single `NewServer` per process or your own `Registerer`.

```go
import httpsrv "github.com/DjaPy/gokit-services/pkg/http/server"

mux := http.NewServeMux()
mux.HandleFunc("GET /health", healthHandler)

srv := httpsrv.NewServer(mux,
    httpsrv.WithPort(8080),
    httpsrv.WithAppName("my-svc"),
)
```

A panic in a handler returns an RFC 7807 Problem JSON (`application/problem+json`) with status 500 to the client — only if the response hasn't started being sent yet.

`responseWriter` forwards `http.Flusher` and `http.Hijacker` — SSE and WebSocket work correctly.

## http/client

HTTP client with a fixed base URL and a middleware chain. The generic `Do[T]` decodes a JSON response into T.

```go
import httpcli "github.com/DjaPy/gokit-services/pkg/http/client"

c, err := httpcli.New("https://api.example.com",
    httpcli.WithTimeout(10 * time.Second),
    httpcli.WithMiddleware(authMiddleware, tracingMiddleware),
)

type User struct { Name string `json:"name"` }
user, err := httpcli.Do[User](ctx, c, http.MethodGet, "/users/42")
```

**`Do[T]`** returns an error on non-2xx. For an empty body (204 No Content) it returns the zero value without an error.

**Middleware:** the first in the list is the outermost (runs first). It is applied after all Options, so `WithMiddleware` and `WithTransport` can be passed in any order.

**`WithBody`** sets `Content-Length` automatically for types implementing `Len() int` (`*bytes.Buffer`, `*strings.Reader`).

## grpc/server, grpc/client

Both collect Prometheus metrics out of the box through built-in unary and stream interceptors,
prepended (outermost) so any interceptors added via `WithServerOptions` / `WithDialOptions` are
still timed and counted. Configure with `WithAppName` and `WithPrometheusRegisterer` (as with
`http/server`, pass `prometheus.NewRegistry()` in tests — `internal/prom.RegisterOrReuse` also
makes repeated construction on the default registerer safe).

**Metrics:** `grpc_server_handled_total`, `grpc_server_handling_seconds`,
`grpc_server_requests_inflight` (server) and the matching `grpc_client_*` (client). Labels:
`grpc_service` (app name), `grpc_type` (`unary` / `client_stream` / `server_stream` /
`bidi_stream`), `grpc_method` (full RPC method), `grpc_code` (status code; inflight gauge omits
it). Server-stream duration is the handler's lifetime; client streams are observed on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DjaPy/gokit-services](https://github.com/DjaPy/gokit-services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->

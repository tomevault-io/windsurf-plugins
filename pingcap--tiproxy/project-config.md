---
trigger: always_on
description: This file provides guidance to agents (human and AI) when working with code in this repository.
---


# AGENTS.md

This file provides guidance to agents (human and AI) when working with code in this repository.

The goal is to:

- Keep code structure consistent.
- Make changes safe and testable.
- Avoid common pitfalls around concurrency, logging, and configuration.

## Development Environment

- **Go version**
  - Use the Go version specified in `go.mod` or the CI configuration.
  - Do not downgrade or upgrade the Go version casually; if a version change is needed, treat it as a dedicated change and update CI accordingly.

- **Module and dependencies**
  - Use `go mod tidy` to manage dependencies; do not edit `go.sum` manually.
  - Avoid committing temporary or experimental dependencies; prefer small, focused changes that clearly justify new modules.

- **Formatting and linting**
  - Always run `make lint` after your changes.

## Code Organization

When adding or modifying features, prefer extending existing packages before creating new ones. If a new package is required, document it here so future contributors understand its purpose.

**Package structure:**

- `cmd/replayer/` - Entry point for the client-mode traffic replayer.
- `cmd/tiproxy/` - Entry point for the TiProxy service.
- `cmd/tiproxyctl/` - Entry point for the controller that operates TiProxy based on the TiProxy API.
- `lib/cli/` - Implementation for `tiproxyctl`.
- `lib/config/` - Definition and default values of TiProxy configurations.
- `lib/util/` - Utilities, including error wrapping, logging helpers, retrial process, certification, etc.
- `pkg/balance/factor/` - Factor-based load-balance policy. The load balancer calculates backend scores based on these factors.
- `pkg/balance/metricsreader/` - Collects backend metrics so that factors can calculate scores based on metrics.
- `pkg/balance/observer/` - Queries backend list and checks backend health, then notifies the router.
- `pkg/balance/policy/` - The `BalancePolicy` interface and implementations such as `SimpleBalancePolicy`.
- `pkg/balance/router/` - Load-balance and routing service. Rebalances and routes connections based on a `BalancePolicy`.
- `pkg/manager/cert/` - Auto-reloads certificates and provides interfaces to query them.
- `pkg/manager/config/` - Auto-reloads configuration files and provides interfaces to query them.
- `pkg/manager/elect/` - Manages TiProxy owner elections (for example, metrics reader and VIP modules need an owner).
- `pkg/manager/id/` - Generates global IDs.
- `pkg/manager/backendcluster/` - Manages cluster-scoped backend runtimes and shared resources such as PD or etcd clients.
- `pkg/manager/infosync/` - Queries the topology of TiDB and Prometheus from PD and updates TiProxy information to PD.
- `pkg/manager/logger/` - Manages the logger service.
- `pkg/manager/memory/` - Records heap and goroutine profiles when memory usage is high.
- `pkg/manager/meter/` - Outputs outbound and cross-AZ network bytes to external storage.
- `pkg/manager/namespace/` - Manages multiple namespaces.
- `pkg/manager/vip/` - Elects, binds, and unbinds the virtual IP address for high availability.
- `pkg/metrics/` - Defines Prometheus metrics.
- `pkg/proxy/` - Starts and stops the proxy service.
- `pkg/proxy/backend/` - Handshakes and forwards messages between clients and backends; also migrates connections among backends if the load balancer requests it.
- `pkg/proxy/client/` - Wrapper of the client connection.
- `pkg/proxy/keepalive/` - Configures TCP keep-alive settings.
- `pkg/proxy/net/` - Parses and packages MySQL-wire packets.
- `pkg/proxy/proxyprotocol/` - Parses and fills the Proxy Protocol header.
- `pkg/sctx/` - Defines the global context.
- `pkg/server/` - Starts and stops the TiProxy server, including proxy service, API service, and other managers.
- `pkg/server/api/` - Starts and stops the TiProxy API service.
- `pkg/sqlreplay/` - Captures and replays traffic.
- `pkg/testkit/` - Common test utilities.
- `pkg/util/` - Utilities, including buffered IO, etcd wrapper, HTTP wrapper, simple MySQL lexer, and others.

Guidelines:

- New load-balance factors should go to `pkg/balance/factor/`.
- New management logic that needs coordination or auto-reload should go under the corresponding `pkg/manager/*` package where possible.
- Shared helpers should go to `lib/util` or `pkg/util` instead of being duplicated across packages.
- After adding a new top-level package or command, update this file to describe it.

## Code Style

- **File headers**
  - When creating new source files (for example `*.go`), include the TiProxy copyright and Apache 2.0
    license header at the top.
  - Copy the header from an existing file in the same directory and update the year if needed.

- **Structs and pointers**
  - Use pointers for large structs or structs that are frequently passed around.
  - Avoid pointer-to-interface patterns; prefer passing interfaces by value.
  - Avoid copying structs that contain mutexes or other synchronization primitives; instead, pass pointers or provide methods on the owning type.

- **Error handling**
  - Prefer wrapping errors with context so they are actionable (what operation failed, which component, key parameters).
  - Reuse common error helpers from `lib/util` / `pkg/util` when available.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pingcap/tiproxy](https://github.com/pingcap/tiproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

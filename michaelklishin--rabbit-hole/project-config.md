---
trigger: always_on
description: This is `rabbit-hole`, a Go client for the [RabbitMQ HTTP API](https://www.rabbitmq.com/docs/management#http-api).
---

# Instructions for AI Agents

## What is This Codebase?

This is `rabbit-hole`, a Go client for the [RabbitMQ HTTP API](https://www.rabbitmq.com/docs/management#http-api).
It provides a single `Client` type with methods for managing and monitoring RabbitMQ clusters.
See [CONTRIBUTING.md](./CONTRIBUTING.md) for development setup details.

## Build

```shell
go build ./...
go vet ./...
go fmt ./...
```

## Run Tests

This library uses [Ginkgo](https://onsi.github.io/ginkgo/) for tests.

```shell
# run all tests (requires a running RabbitMQ node)
go test -v
```

Always run `go build ./...` and `go vet ./...` before making changes to verify the codebase compiles
and passes static analysis. If either fails, investigate and fix errors before proceeding with any modifications.

At the end of each task, run `go fmt ./...`.

### Test Node Setup

Tests are integration tests that require a locally running RabbitMQ node with the management
plugin enabled and specific virtual hosts, users, and plugins configured.

#### Use a Container

The following Make target starts a fully configured node in a container:

```shell
gmake docker.rabbitmq

# In a separate shell
go test -v
```

`gmake docker.rabbitmq` creates the `rabbit/hole` virtual host, the `policymaker` user, and enables
the `rabbitmq_federation`, `rabbitmq_federation_management`, `rabbitmq_shovel`, and
`rabbitmq_shovel_management` plugins.

#### BYON (Bring Your Own Node)

Alternatively, start a local RabbitMQ node any way you like, then run `bin/ci/before_build.sh` that will
set it up:

```shell
./bin/ci/before_build.sh
go test -v
```

This script enables the required plugins, reduces the stats emission interval, and creates the
virtual hosts and users the test suite expects. It requires `rabbitmqctl` to be in `PATH`, or the
`RABBITHOLE_RABBITMQCTL` environment variable to point to it.

## Repository Layout

All library code lives at the top level of the repository (single package, no subdirectories):

 * `client.go`: `Client` type, constructor functions (`NewClient`, `NewTLSClient`), request helpers
 * `error.go`: `ErrorResponse` type and error handling
 * `common.go`: shared data types and custom JSON unmarshalers
 * `doc.go`: package-level documentation with usage examples
 * Domain-specific files by resource type:
   * `vhosts.go`, `vhost_limits.go`, `vhosts_channels.go`, `vhosts_connections.go`
   * `users.go`, `permissions.go`, `topic_permissions.go`, `user_limits.go`
   * `exchanges.go`, `queues.go`, `bindings.go`, `consumers.go`
   * `policies.go`, `operator_policies.go`
   * `nodes.go`, `cluster.go`, `health_checks.go`
   * `federation.go`, `federation_links.go`, `shovels.go`, `streams.go`
   * `runtime_parameters.go`, `global_parameters.go`, `feature_flags.go`
   * `definitions.go`, `plugins.go`, `deprecated_features.go`, `misc.go`
 * `*_test.go`: integration and unit tests using [Ginkgo v2](https://onsi.github.io/ginkgo/) and [Gomega](https://onsi.github.io/gomega/)
 * `go.mod`: module definition (`github.com/michaelklishin/rabbit-hole/v3`)
 * `Makefile`: common development targets
 * `bin/ci/before_build.sh`: configures an existing RabbitMQ node for the test suite (plugins, vhosts, users, stats interval)

## Code Conventions

### Naming

 * Types: `PascalCase` — `VhostInfo`, `QueueSettings`, `ExchangeInfo`
 * `*Info` suffix for read-only API response structs (e.g. `QueueInfo`, `NodeInfo`)
 * `*Settings` suffix for mutable request structs used to create or update resources (e.g. `QueueSettings`, `VhostSettings`)
 * Client methods: verb-first `PascalCase` — `ListQueues`, `GetExchange`, `DeclareQueue`, `DeleteBinding`
 * JSON struct tags: `snake_case` (matching RabbitMQ HTTP API conventions)

### Operation Structure

Each resource domain follows a consistent pattern:

```go
// List/Get — HTTP GET, returns parsed response
func (c *Client) ListQueues() (rec []QueueInfo, err error)
func (c *Client) GetQueue(vhost, queue string) (rec *QueueInfo, err error)

// Create/Update — HTTP PUT with JSON body, returns raw *http.Response
func (c *Client) DeclareQueue(vhost string, queue string, info QueueSettings) (res *http.Response, err error)

// Delete — HTTP DELETE, returns raw *http.Response; 404 is treated as success (idempotent)
func (c *Client) DeleteQueue(vhost, queue string) (res *http.Response, err error)
```

### Custom JSON Unmarshaling

The library handles RabbitMQ API quirks with custom unmarshalers for types that can appear
as multiple JSON types: `Port` (int, string, or `"undefined"`), `URISet` (string or array),
`AutoDelete` (bool or `"undefined"`), `UserTags` (comma-separated string or array).
Follow the same pattern in `common.go` when adding new fields with ambiguous types.

## Test Suite Layout

 * `rabbithole_suite_test.go`: Ginkgo suite bootstrap (`TestRabbitHole`) and shared setup
 * `rabbithole_test.go`: main integration test suite (requires a running RabbitMQ node)
 * `unit_test.go`: Ginkgo specs for custom JSON unmarshaling; can be isolated with `--ginkgo.focus="Unit tests"`
 * `health_checks_test.go`: health check endpoint tests
 * `policies_test.go`: policy and operator policy tests
 * `deprecated_features_test.go`: Ginkgo specs for `DeprecationPhase` JSON unmarshaling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michaelklishin/rabbit-hole](https://github.com/michaelklishin/rabbit-hole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

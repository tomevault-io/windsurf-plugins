---
trigger: always_on
description: kafkactl is a command-line interface for Apache Kafka written in Go. It supports topic/consumer-group/ACL/broker/user management, message producing/consuming with Avro/Protobuf/JSON schema support, Kubernetes-proxied execution, and an external plugin system for OAuth token providers.
---

# CLAUDE.md - kafkactl Project Guide

## Project Overview

kafkactl is a command-line interface for Apache Kafka written in Go. It supports topic/consumer-group/ACL/broker/user management, message producing/consuming with Avro/Protobuf/JSON schema support, Kubernetes-proxied execution, and an external plugin system for OAuth token providers.

- **Module**: `github.com/deviceinsight/kafkactl/v5`
- **Go version**: 1.24.12
- **Kafka client**: IBM/sarama
- **CLI framework**: spf13/cobra + spf13/viper
- **License**: Apache 2.0

## Build & Run Commands

```bash
# Build the binary
make build

# Run all checks (fmt, lint, cve-check, test, build, docs)
make all

# Format code
make fmt                    # runs gofmt + goimports

# Lint (golangci-lint via Go tool directive)
make lint                   # runs: go tool golangci-lint run

# Unit tests only (skips integration tests)
make test                   # runs: go tool gotestsum --format testname --hide-summary=skipped -- -v -short ./...

# Integration tests (requires Docker)
make integration_test       # starts docker-compose cluster, runs tests, tears down

# CVE check
make cve-check              # runs: go tool govulncheck ./...

# Clean
make clean
```

**Important**: CGO is disabled (`CGO_ENABLED=0`). Tools (golangci-lint, goimports, govulncheck, gotestsum) are managed via `go.mod` `tool` directives, invoked with `go tool <name>`.

## Testing

### Test Naming Convention

- **Unit tests**: Standard Go test functions (e.g., `TestEnvironmentVariableLoading`). No special suffix required.
- **Integration tests**: **Must** be suffixed with `Integration` (e.g., `TestGetTopicsIntegration`). This is enforced by `StartIntegrationTest()` which fatals if the test name doesn't end with `Integration`.

### Running Tests

```bash
# Unit tests only (integration tests are skipped via -short flag)
go test -v -short ./...

# Integration tests (requires Docker Kafka cluster running first)
cd docker && docker compose up -d
go test -v -run Integration ./...

# Stop integration test cluster when done
cd docker && docker compose down
```

### Integration Test Infrastructure

The `docker/docker-compose.yml` provides:
- 3 Kafka brokers (Confluent Platform) with SASL PLAIN, SCRAM-SHA-256, SCRAM-SHA-512
- 1 ZooKeeper
- 1 Schema Registry (Avro + JSON Schema)
- ACL authorizer enabled with `admin` as super user

Test contexts defined in `it-config.yml`:
- `default` - plaintext, ports 19093/29093/39093
- `no-schema-reg` - plaintext, no schema registry
- `sasl-admin` - SASL PLAIN as admin, ports 19092/29092/39092
- `sasl-user` - SASL PLAIN as user
- `k8s-mock` - Kubernetes mock via `docker/kubectl-mock.sh`
- `scram-admin` - SCRAM-SHA-256, ports 19094/29094/39094

### Test Utilities (`internal/testutil/`)

- `StartUnitTest(t)` - call at the start of every unit test
- `StartIntegrationTest(t)` / `StartIntegrationTestWithContext(t, "context")` - call at the start of integration tests
- `CreateKafkaCtlCommand()` - creates a test command instance with captured stdout/stderr
- `CreateTopic(t, prefix, flags...)` - creates a topic with a random suffix to avoid collisions
- `CreateConsumerGroup(t, prefix, topics...)` - creates a consumer group
- `ProduceMessage(t, topic, key, value, partition, offset)` - produces a test message
- `AssertEquals(t, expected, actual)`, `AssertContains`, `AssertErrorContains`, etc. - custom assertions (not using testify)
- `GetPrefixedName(prefix)` - generates `prefix-<random>` names to prevent test interference
- `WithoutBrokerReferences(output)` - normalizes broker IDs/addresses for stable test output

Tests do **not** clean up resources. Random suffixes on topic/group names prevent collisions.

### Test Framework

Uses standard `testing` package with custom assertion helpers in `internal/testutil/`. Does **not** use testify. Tests are run directly with `go test`.

## Project Architecture

### Directory Structure

```
kafkactl/
  main.go                     # Entry point, signal handling, shutdown timeout
  cmd/                        # CLI command definitions (cobra commands)
    root.go                   # Root command, registers all subcommands
    alter/                    # alter topic|partition|broker|user
    attach/                   # attach (K8s interactive shell)
    clone/                    # clone topic|consumer-group
    config/                   # config current-context|get-contexts|use-context|view
    consume/                  # consume <topic>
    create/                   # create topic|consumer-group|acl|user
    deletion/                 # delete topic|consumer-group|offset|records|acl|user
    describe/                 # describe topic|consumer-group|broker|user
    get/                      # get topics|consumer-groups|acl|brokers|users
    produce/                  # produce <topic>
    reset/                    # reset offset
    validation/               # Custom flag validation (at-least-one-required)
    completion.go             # Shell completion command

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deviceinsight/kafkactl](https://github.com/deviceinsight/kafkactl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

---
trigger: always_on
description: Gnetcli is a Go project for automating interactive network-device CLIs. It contains:
---

# AGENTS.md

## Project overview

Gnetcli is a Go project for automating interactive network-device CLIs. It contains:

- Go library APIs under `pkg/`.
- CLI binaries under `cmd/`:
  - `cmd/cli` - command-line client.
  - `cmd/gnetcli_server` - gRPC/HTTP gateway server.
  - `cmd/gswitch` and `cmd/gvendor` - auxiliary tools.
- Device implementations under `pkg/device/*` for vendors such as Huawei, Cisco, Juniper, NX-OS, RouterOS, FortiOS, Arista, H3C, Eltex, ASA, Netconf, and generic CLI.
- Transport/connectors under `pkg/streamer/*` for SSH, telnet, console, RFC2217, and common streamer abstractions.
- gRPC server implementation and generated protobuf files under `pkg/server` and `pkg/server/proto`.
- Python gRPC SDK under `grpc_sdk/python`.
- Documentation sources under `docs/`, built with MkDocs into `site/`.
- Examples under `examples/` and benchmark code under `benchmarks/`.

Core architecture: high-level `device.Device` implementations execute `pkg/cmd.Cmd` over low-level `streamer.Connector` transports. Most vendor support is built on `pkg/device/genericcli`, which uses regular expressions from `pkg/expr` to detect prompts, errors, pagers, questions, login/password prompts, and command echo.

## Repository conventions

- Go module: `github.com/annetutil/gnetcli`.
- Go version in `go.mod`: `1.25`.
- Keep Go code formatted with `gofmt`/`go test` conventions.
- Use existing functional-option style for constructors and configuration (`With...` options).
- Prefer adding behavior through public interfaces and options instead of reaching into private fields.
- Tests use the standard Go `testing` package plus `github.com/stretchr/testify` where already present.
- Mock CLI/device interaction tests should follow `pkg/testutils/mock` patterns used in `pkg/device/*/*_test.go`.
- Python SDK style is governed by `grpc_sdk/python/pyproject.toml` and `tox.ini`:
  - Ruff line length is 120.
  - Flake8 max line length is 140.
  - Generated `.pb2.py` and `.pb2_grpc.py` are excluded from flake8.

## Important files and packages

- `pkg/device/device.go` - main `Device` interface.
- `pkg/device/genericcli/genericcli.go` - regex-driven generic CLI implementation and device options.
- `pkg/cmd/cmd.go` - command and command-result interfaces, command options, question handling.
- `pkg/streamer/streamer.go` - `Connector` interface and common read/write/file abstractions.
- `pkg/expr/expr.go` - expression matching helpers.
- `pkg/server/proto/server.proto` - gRPC API definition.
- `pkg/server/*.go` - server business logic.
- `cmd/gnetcli_server/server.go` - server entry point, auth, TLS, gRPC/HTTP setup.
- `grpc_sdk/python/gnetclisdk/` - Python client SDK.
- `Makefile` - main build/test/protobuf commands.
- `docs/architecture.md` - architecture and device-development guidance.
- `docs/dev.md` - docs and Python package build notes.

## Build and test commands

Use the narrowest relevant command while iterating, then run broader checks before finishing.

### Go

```sh
go test ./...
```

Race tests used by the project Makefile:

```sh
make testrace
```

Full project target:

```sh
make all
```

`make all` runs build, protobuf generation, and race tests. It requires Docker and a local `proto_builder:tag` image for the `proto` target.

### Protobuf generation

Build the protobuf builder image first if needed:

```sh
make build-proto-docker
```

Regenerate protobuf outputs:

```sh
make proto
```

Generated files live in `pkg/server/proto/` and include Go, grpc-gateway, and Python artifacts. Do not manually edit generated protobuf outputs unless explicitly requested.

### Docker

```sh
make build-docker
```

or:

```sh
docker build -f image/Dockerfile -t gnetcli-server .
```

### Python SDK

From `grpc_sdk/python`:

```sh
tox -e ci
```

Package build:

```sh
make build
```

Publishing targets exist (`publish-test`, `publish-prod`), but do not run them unless explicitly requested.

### Documentation

Docs are MkDocs sources in `docs/`. See `docs/dev.md` for the Docker-based build flow. The generated `site/` directory is build output.

## Development guidance

### Adding or changing a device vendor

1. Prefer implementing a vendor with `genericcli.MakeGenericCLI` unless the protocol is fundamentally different.
2. Define precise regexes for:
   - prompt detection,
   - command errors,
   - pager prompts,
   - questions/confirmations,
   - login/password prompts when transport-level auth is not enough.
3. Use `expr.NewSimpleExprLast200().FromPattern(...)` for prompt/error/pager/question expressions when matching terminal tail output.
4. Add auto-commands with `genericcli.WithAutoCommands` only when the device commonly needs session setup, and make them tolerate unsupported commands with `cmd.WithErrorIgnore()` where appropriate.
5. Add tests with `pkg/testutils/mock` dialogs that cover:
   - successful command execution,
   - invalid command/error parsing,
   - pager handling if supported,
   - questions/answers if supported,
   - login edge cases if relevant.
6. Do not rely on a real network device in unit tests.

### Working with command execution

- Prefer `Device.ExecuteCtx` for new code; `Execute` is marked legacy in the interface.
- Preserve command output semantics:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [annetutil/gnetcli](https://github.com/annetutil/gnetcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->

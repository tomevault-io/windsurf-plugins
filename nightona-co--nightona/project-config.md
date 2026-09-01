---
trigger: always_on
description: This project uses **Nix flakes** to provide reproducible development environments outside of the devcontainer. When working on this codebase, use the appropriate Nix dev shell to ensure all build tools, language runtimes, and dependencies are available.
---

# Agent Development Guide

This project uses **Nix flakes** to provide reproducible development environments outside of the devcontainer. When working on this codebase, use the appropriate Nix dev shell to ensure all build tools, language runtimes, and dependencies are available.

## Prerequisites

- **Nix** with flakes enabled — [install guide](https://nixos.org/download/)
- Enable flakes: add `experimental-features = nix-command flakes` to `~/.config/nix/nix.conf`

## Available Dev Shells

| Shell | Command | Languages / Tools |
|---|---|---|
| `default` | `nix develop` | Go + Node.js + Python + Ruby + JDK (everything) |
| `go` | `nix develop .#go` | Go, golangci-lint, protobuf, buf |
| `node` | `nix develop .#node` | Node.js 22, Yarn 4 (via corepack) |
| `python` | `nix develop .#python` | Python 3.12, Poetry |
| `ruby` | `nix develop .#ruby` | Ruby 3.4, Bundler |
| `java` | `nix develop .#java` | JDK 17, Gradle |

## Running Commands in Nix Shells

### Interactive (human use)

```bash
nix develop .#go    # drops you into a shell with Go tools
```

### Non-interactive (agent / CI use)

Use `--command` to run a single command inside the shell and exit:

```bash
nix develop .#go     --command bash -c "go build ./apps/cli/..."
nix develop .#node   --command bash -c "yarn install && yarn build"
nix develop .#python --command bash -c "poetry install && poetry run pytest"
nix develop .#ruby   --command bash -c "bundle install && bundle exec rspec"
```

For short commands you can also use:

```bash
nix develop .#go --command go test ./libs/sdk-go/...
nix develop .#go --command golangci-lint run ./apps/runner/...
```

## Project → Shell Mapping

Use this table to determine which shell to enter for a given directory.

### Go projects → `nix develop .#go`

| Directory | Description |
|---|---|
| `apps/cli` | CLI application |
| `apps/daemon` | Background daemon service |
| `apps/proxy` | Network proxy |
| `apps/runner` | Code execution service |
| `apps/snapshot-manager` | Snapshot management |
| `apps/ssh-gateway` | SSH gateway |
| `apps/otel-collector/exporter` | OpenTelemetry exporter |
| `libs/sdk-go` | Go SDK |
| `libs/api-client-go` | Go API client |
| `libs/common-go` | Shared Go utilities |
| `libs/computer-use` | Computer use library |
| `libs/toolbox-api-client-go` | Go toolbox API client |

All Go modules are coordinated via `go.work` at the repo root.

### Node.js / TypeScript projects → `nix develop .#node`

| Directory | Description | Build |
|---|---|---|
| `apps/api` | NestJS backend API | `npx nx build api` (Webpack) |
| `apps/dashboard` | React SPA dashboard | `npx nx build dashboard` (Vite) |
| `apps/docs` | Astro documentation site | `npx nx build docs` |
| `libs/sdk-typescript` | TypeScript SDK | `npx nx build sdk-typescript` |
| `libs/api-client` | TypeScript API client | `npx nx build api-client` |
| `libs/toolbox-api-client` | TypeScript toolbox API client | `npx nx build toolbox-api-client` |
| `libs/analytics-api-client` | Analytics API client | `npx nx build analytics-api-client` |
| `libs/runner-api-client` | Runner API client | `npx nx build runner-api-client` |
| `libs/opencode-plugin` | OpenCode plugin | `npx nx build opencode-plugin` |

All TS/Node projects are managed via **Nx** with **Yarn 4** workspaces.

### Python projects → `nix develop .#python`

| Directory | Description |
|---|---|
| `libs/sdk-python` | Python SDK |
| `libs/api-client-python` | Python API client |
| `libs/api-client-python-async` | Python async API client |
| `libs/toolbox-api-client-python` | Python toolbox API client |
| `libs/toolbox-api-client-python-async` | Python async toolbox API client |
| `examples/python` | Python SDK examples |
| `guides/python` | Python guides |

All Python packages are managed via **Poetry** from the root `pyproject.toml`.

### Ruby projects → `nix develop .#ruby`

| Directory | Description |
|---|---|
| `libs/sdk-ruby` | Ruby SDK |
| `libs/api-client-ruby` | Ruby API client |
| `libs/toolbox-api-client-ruby` | Ruby toolbox API client |
| `examples/ruby` | Ruby SDK examples |

Ruby gems are managed via **Bundler** using the root `Gemfile`.

### Java projects → `nix develop .#java`

| Directory | Description |
|---|---|
| `libs/sdk-java` | Java SDK |
| `libs/api-client-java` | Java API client (auto-generated) |
| `libs/toolbox-api-client-java` | Java toolbox API client (auto-generated) |
| `examples/java` | Java SDK examples |

All Java projects use **Gradle** (Kotlin DSL) with the Gradle wrapper.

## Common Build & Test Commands

### Go

```bash
# Build all Go modules
nix develop .#go --command bash -c "go build ./..."

# Build a specific app
nix develop .#go --command bash -c "go build ./apps/runner/..."

# Run tests for a specific module
nix develop .#go --command bash -c "go test ./libs/sdk-go/..."

# Run tests for all Go modules
nix develop .#go --command bash -c "go test ./..."

# Lint
nix develop .#go --command bash -c "golangci-lint run ./apps/runner/..."

# Generate swagger docs
nix develop .#go --command bash -c "swag init -g apps/daemon/cmd/main.go"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nightona-co/nightona](https://github.com/nightona-co/nightona) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->

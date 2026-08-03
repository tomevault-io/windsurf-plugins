---
trigger: always_on
description: This document helps AI assistants understand the codebase and contribute effectively.
---

# CLAUDE.md - unpush

This document helps AI assistants understand the codebase and contribute effectively.

## Project overview

`unpush` is a continuous deployment service for [Uncloud](https://github.com/psviderski/uncloud). It runs as a container inside an Uncloud cluster and deploys services automatically when a branch changes, triggered by GitHub push webhooks or by polling the remote branch on a configurable interval.

The deployer connects to the Uncloud daemon through its Unix socket (`/run/uncloud/uncloud.sock`), which every node in the cluster exposes. This gives the deployer full cluster access without needing SSH keys or network configuration.

## Repository structure

```
main.go         Entry point. Loads config, opens state DB, registers webhook routes, runs the HTTP server.
config.go       Loads AppConfig from a YAML file (UNPUSH_CONFIG, default /deploy/config.yaml).
webhook.go      GitHub webhook handler. Reads body, verifies HMAC, dispatches to deployer.
deployer.go     Core deploy logic. Connects to socket, loads compose file, plans and executes deploy.
build.go        Builds services with a build directive and pushes images to cluster machines.
repo.go         Clones or fetches the repository at the push commit (repo mode only).
state.go        SQLite state database. Records every deploy attempt and outcome per target.
poller.go       Poll trigger. Checks remote HEAD on an interval and retries failed deploys.
Dockerfile      Multi-stage build. Build context is the deployer directory.
compose.yaml    Reference compose file for deploying the deployer itself into Uncloud.
mise.toml       Build tooling. Go 1.26.1 via mise. Tasks: build, run, build:image.
misc/design.md  Architecture decisions and options considered during design.
```

## Key types

`AppConfig` — top-level config with `ListenAddr`, `StateDB`, and `Targets []TargetConfig`.

`TargetConfig` — per-target settings: `Name`, `WebhookSecret`, `Branch`, `ComposeFile`, `ForceRecreate`, `RepoURL`, `RepoToken`, `WorkDir`, `PollInterval`, `EnableWebhook`, `SocketPath`. Each `Deployer` holds one `TargetConfig`.

`Deployer` — holds a `TargetConfig`, a shared `*sql.DB`, and a buffered channel queue (capacity 1).

## Configuration

Configuration is always loaded from a YAML file. `loadAppConfig` reads the path from `UNPUSH_CONFIG`, defaulting to `/deploy/config.yaml`. `loadFileConfig` parses the file and fills in defaults: `branch` → `main`, `work_dir` → `/deploy/work/<name>`, `compose_file` → `compose.yaml` if `repo_url` is set, `/deploy/compose.yaml` otherwise, `state_db` → `/deploy/state.db`, `enable_webhook` → `true`. It also validates that every target has a unique non-empty name and copies the global `socket_path` into each `TargetConfig`.

Each target registers a webhook handler at `/webhook/<name>` by default. Set `enable_webhook: false` to skip registration (requires `poll_interval`). Poll and webhook triggers can be active simultaneously on the same target.

## Key dependencies

The deployer imports `github.com/psviderski/uncloud` as a standard Go module dependency pinned to a specific commit. The uncloud packages used directly:

| Package                | Purpose                                                                   |
| ---------------------- | ------------------------------------------------------------------------- |
| `pkg/client`           | `client.New` creates a cluster client from a connector                    |
| `pkg/client/connector` | `NewUnixConnector` connects to the daemon socket                          |
| `pkg/client/compose`   | `LoadProject` and `NewDeploymentWithStrategy` implement `uc deploy` logic |
| `pkg/client/deploy`    | `RollingStrategy` controls how containers are updated                     |

In repo mode, the deployer also uses these directly (both are transitive dependencies of uncloud):

| Package                                            | Purpose                                                      |
| -------------------------------------------------- | ------------------------------------------------------------ |
| `github.com/docker/cli/cli/command`                | Creates a Docker CLI client for the build step               |
| `github.com/docker/compose/v2/pkg/compose`         | Builds images via the Compose Go library                     |
| `github.com/google/go-containerregistry/pkg/crane` | Pushes images to remote machine unregistries over plain HTTP |
| `modernc.org/sqlite`                               | Pure-Go SQLite driver (no cgo) for the deploy state database |

`internal/cli.BuildServices` in uncloud contains equivalent build logic but is not importable from outside the module. `build.go` replicates the relevant parts. See the TODO comment there for the long-term option.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tonyo/unpush](https://github.com/tonyo/unpush) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

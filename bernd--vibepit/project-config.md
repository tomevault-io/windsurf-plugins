---
trigger: always_on
description: manages persistent `vibepit-home` volume and per-session networking.
---

## Project

Vibepit runs development agents inside isolated Docker/Podman containers with
network isolation via a filtering proxy. A single `vibepit` command launches a
proxy container and a sandbox container on an isolated network, with a persistent
home volume, the project directory mounted in, and a security-hardened runtime
(read-only root filesystem, dropped capabilities, no-new-privileges).

## Prerequisites

- Go installed locally (project currently uses Go 1.26.x in CI/dev).
- Docker or Podman installed and running (host development only).
- Your user can access the container runtime socket (host development only).
- For release tasks: `gh` CLI authenticated and a valid git tag.

## Nested Sandbox Context (Vibepit-in-Vibepit)

When you use Vibepit to develop Vibepit itself, expect these constraints:

- No Docker/Podman socket or API access from inside the sandbox.
- No direct internet access except explicitly allowlisted destinations.
- `vibepit` runtime commands that need container/network control are expected to
  fail in this environment.

In this context, prefer code-level validation (`make test`,
`make test-integration`, targeted `go test`) over attempting runtime session
operations.

## Build, Run, and Test

Use `go run` for local execution to avoid leaving build artifacts. In nested
sandbox development, treat runtime command execution as optional/manual host
verification.

```bash
go run .                     # default command: run
go run . -L                  # use local image instead of published one
go run . -a example.com:443  # allow additional domain:port
go run . -p vcs-github       # enable additional network preset
go run . --reconfigure       # re-run interactive setup
```

Use Make targets for reproducible build/test workflows:

```bash
make build             # build vibepit binary for current platform
make test              # run unit tests
make test-integration  # run integration tests (60s timeout)
make test-bats         # run BATS tests for image/entrypoint scripts
make clean             # remove binary and dist/ artifacts
```

## CLI Command Reference

Current root commands are defined in `cmd/root.go` and include:

- `run` (default) -- launch or attach to a sandbox session.
- `allow-http` -- add HTTP(S) allowlist entries at runtime.
- `allow-dns` -- add DNS allowlist entries at runtime.
- `proxy` -- internal command used inside the proxy container.
- `sessions` -- list active sessions.
- `monitor` -- interactive TUI for logs and allowlist/admin actions.
- `update` -- pull latest runtime images.

When docs and behavior differ, treat `cmd/root.go` and command files under
`cmd/` as the source of truth.

## Architecture

### Go CLI (`cmd/`)

Built with `urfave/cli/v3`.

- `run` creates an isolated network, starts proxy + sandbox containers, and
  manages persistent `vibepit-home` volume and per-session networking.
- `allow-http` / `allow-dns` call the control API and can persist to project
  config.
- `proxy` runs the proxy server inside the proxy container.
- `sessions` and `monitor` provide session discovery and interactive control.
- `update` refreshes local runtime images.

### Proxy (`proxy/`)

Network isolation layer running three services in the proxy container:

1. HTTP proxy (dynamic port) filtering HTTP/HTTPS via allowlist rules.
2. DNS server (port 53) filtering DNS queries via allowlist rules.
3. mTLS-secured control API (dynamic port) for runtime admin and logs.

Key files:
- `proxy/server.go` -- service orchestration.
- `proxy/http.go` -- HTTP filtering.
- `proxy/dns.go` -- DNS filtering.
- `proxy/allowlist.go` -- domain/port matching.
- `proxy/cidr.go` -- IP range blocking.
- `proxy/api.go` -- control API.
- `proxy/mtls.go` -- cert generation/validation.
- `proxy/log.go` -- request log buffer.
- `proxy/presets.go` / `proxy/presets.yaml` -- network presets.

### Container client (`container/`)

Docker/Podman client abstractions for networks, containers, attach/exec, and
volumes. Terminal I/O handling lives in `container/terminal.go`.

### Configuration (`config/`)

YAML project config via `knadh/koanf`, including first-run/reconfigure setup UI
and runtime detection helpers.

### TUI (`tui/`)

Bubble Tea-based terminal UI primitives for window framing, header/cursor, and
screen abstraction.

### Embedded proxy binary (`embed/`)

Used during release builds to embed the Linux arm64 proxy binary for macOS
runtime compatibility via Go embed.

### Container image (`image/`)

- `image/Dockerfile` -- Ubuntu base + dev tools, non-root `code` user
  (`CODE_UID`/`CODE_GID` build args).
- `image/entrypoint.sh` -- initializes home template and starts shell.
- `image/entrypoint-lib.sh` -- shared shell library sourced by `entrypoint.sh`.
- `image/lib.sh` -- additional shell library.
- `image/config/` -- shell and tool configs: `profile`, `bashrc`,
  `bashrc.tail`, `tmux.conf`.
- `image/bin/*` -- runtime installers (`brew`, `claude`, `ccusage`, `cxusage`,
  `yoloclaude`) and a `skills` helper script, run by users inside container,
  not during image build.
- `image/tests/` -- BATS tests for entrypoint scripts (run via `make test-bats`).

Runtime installers persist in the home volume and are intentionally not baked
into the base image.

## Development Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bernd/vibepit](https://github.com/bernd/vibepit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->

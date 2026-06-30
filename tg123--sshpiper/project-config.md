---
trigger: always_on
description: sshpiper is a reverse proxy for SSH. It sits between SSH clients ("downstream") and SSH servers ("upstream"), routing connections based on configurable plugins. All SSH-based protocols (ssh, scp, port forwarding) are supported.
---

# Copilot Instructions for sshpiper

## What is sshpiper

sshpiper is a reverse proxy for SSH. It sits between SSH clients ("downstream") and SSH servers ("upstream"), routing connections based on configurable plugins. All SSH-based protocols (ssh, scp, port forwarding) are supported.

## Build

```bash
# Build root module (plugins, admin/webadmin CLI, libs)
mkdir -p out
go build -tags full -o out ./...

# Build the sshpiperd daemon (separate module so the forked crypto stays scoped to it)
(cd cmd/sshpiperd && go build -o ../../out/ .)
```

- **`-tags full`** includes all plugins. Without it, plugins are excluded from the build.
- **CGO is not used** — all builds are pure Go (`CGO_ENABLED=0`).
- **Two Go modules:** the root `github.com/tg123/sshpiper` module uses upstream `golang.org/x/crypto`; the nested `cmd/sshpiperd/go.mod` module contains the daemon and has `replace golang.org/x/crypto => github.com/tg123/sshpiper.crypto <tag>` so only the daemon links against the fork. **No git submodule** — the fork is pulled in as a regular Go module dependency.
- GoReleaser handles release builds, multi-arch Docker images, and Snap packages (`.goreleaser.yaml`).

## Test

### Unit tests

```bash
# Root module: plugins, libs, admin/webadmin
go test -v -race -cover -tags full ./...

# Daemon module
(cd cmd/sshpiperd && go test -v -race -cover ./...)

# Run a single test (specify the module/package)
go test -v -tags full -run ^TestName$ ./path/to/package/
```

> The forked `crypto/ssh` library is **not** tested from this repo — its tests live in the
> [`tg123/sshpiper.crypto`](https://github.com/tg123/sshpiper.crypto) repository and run in that repo's own CI.

### E2E tests

E2E tests require Docker Compose and run inside a `testrunner` container:

```bash
cd e2e
docker compose up --build --force-recreate -d

# Or run with exit code from testrunner:
docker compose up --build --exit-code-from testrunner
```

The E2E suite spins up multiple SSH servers (password auth, pubkey auth, old OpenSSH, CA certs, Kubernetes via Kind) and tests each plugin against them.

### Lint and formatting

```bash
# golangci-lint with build tags (errcheck is disabled). Must be run per module.
golangci-lint run --build-tags full -D errcheck
(cd cmd/sshpiperd && golangci-lint run -D errcheck)

# gofumpt formatting check (stricter than gofmt) — CI uses v0.8.0
gofumpt -l .         # list unformatted files (must be empty)
gofumpt -w .         # auto-fix
```

## ⚠️ Required PR gates — DO NOT skip

Before considering any change/PR complete, you **must** verify all of these locally and confirm green on the PR:

1. **`gofumpt -l .` produces empty output** (workflow: `.github/workflows/gofumpt.yml`). `gofmt` is not enough — this repo enforces `gofumpt`.
2. **`go test -v -race -cover -tags full ./...` passes** in the root module, AND `cd cmd/sshpiperd && go test -v -race -cover ./...` passes in the daemon module (workflow: `.github/workflows/test.yml`).
3. **`golangci-lint run --build-tags full -D errcheck` passes for the root module AND `cd cmd/sshpiperd && golangci-lint run -D errcheck` passes for the daemon module** — both modules are linted independently.
4. **`goreleaser release --snapshot --clean` succeeds** for release-affecting changes (Dockerfile, `.goreleaser.yaml`, new plugin binaries).
5. **E2E suite passes** for changes touching the daemon or plugins: `cd e2e && docker compose up --build --exit-code-from testrunner`.

After pushing to a PR, **always check the GitHub Actions results** (`gh pr checks <pr>` or via the GitHub MCP). Do not declare the task done while any required check is failing or pending — push fixes until every gate is green. Common failure: forgetting to run `gofumpt -w .` after edits, or omitting `-tags full` when running tests/lint locally and missing plugin-specific issues.

## Architecture

### Crypto fork

The forked `golang.org/x/crypto` lives in a **separate repository**,
[`tg123/sshpiper.crypto`](https://github.com/tg123/sshpiper.crypto) (branch `v1`, tagged
`vX.Y.Z-sshpiper-YYYYMMDD`). It is pulled in as a regular Go module dependency — there is **no git submodule**.
The fork is scoped to a single Go module so it does not leak into plugins:

- `cmd/sshpiperd/go.mod` (the daemon module) has `replace golang.org/x/crypto => github.com/tg123/sshpiper.crypto <tag>`. Everything compiled into the `sshpiperd` binary uses the forked `ssh` package.
- The root `go.mod` does **not** replace `golang.org/x/crypto`. Every plugin under `plugin/*`, the libs under `libplugin/` / `libadmin/`, and the `sshpiperd-admin` / `sshpiperd-webadmin` CLIs build against upstream `golang.org/x/crypto` and therefore cannot import fork-only symbols.

The key addition in the fork is `ssh/sshpiper.go`, which adds `PiperConfig` and `PiperConn` — the low-level API for intercepting SSH handshakes and piping two independent SSH connections together. Only the daemon module can reach those symbols.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tg123/sshpiper](https://github.com/tg123/sshpiper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

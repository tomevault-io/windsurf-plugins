---
trigger: always_on
description: Practical reference for agents working in `keploy/keploy`. Everything here is
---

# AGENTS.md

Practical reference for agents working in `keploy/keploy`. Everything here is
grounded in the current state of the repo — if something below conflicts with
what you see on disk, trust the disk and update this file.

## What this project is

Keploy is a backend testing tool that records real API + dependency traffic
from a running application and replays it as deterministic tests with mocks.
It intercepts traffic at the network layer using eBPF (Linux) and a userspace
proxy (macOS/Windows), so apps don't need an SDK or code changes. The binary
is a single Go CLI: `keploy`.

- Go module: `go.keploy.io/server/v3`
- Primary entry point: `main.go` → `cli.Root(...)` in `cli/root.go`

## Build, run, lint, test

### Build the binary

```bash
# Standard build (what CI calls "build-no-race")
go build -tags=viper_bind_struct -o keploy .

# Race-enabled build (what CI calls "build" — used as the default in CI matrices)
CGO_ENABLED=1 go build -race -tags=viper_bind_struct -o keploy .
```

The `viper_bind_struct` build tag is required — leaving it off will cause
config fields not to bind correctly at runtime.

Version / Sentry DSN / server URL / GitHub client ID are injected via
`-ldflags` (see `Dockerfile` and `goreleaser.yaml`). For local development
the default `version` is `3-dev`.

### Run locally

Platform support is not uniform — it's gated by how the agent can
intercept traffic on each OS:

| Platform                  | Native binary (app runs on host)                                                                             | Keploy-in-Docker (app runs in Docker) |
| ------------------------- | ------------------------------------------------------------------------------------------------------------ | ------------------------------------- |
| **Linux** (x86_64, arm64) | ✅ Supported — uses eBPF (`pkg/agent/hooks/linux/`). Requires root.                                          | ✅ Supported                          |
| **Windows** (amd64)       | ✅ Supported — uses the WinDivert redirector (`pkg/agent/hooks/windows/`, `libwindows_redirector.a`).        | ✅ Supported                          |
| **Windows** (arm64)       | ❌ Falls through to the `others` stub — `Load()` / `Record()` return "not supported on non-Linux platforms". | ✅ Supported                          |
| **macOS** (amd64, arm64)  | ❌ Same `others` stub — there is **no** native interception path on macOS.                                   | ✅ Supported (only option)            |

- On **macOS** you _cannot_ use keploy natively. You must:
  1. Build the keploy Docker image: `sudo docker image build -t ghcr.io/keploy/keploy:v3-dev .`
  2. Run your application inside Docker (usually via `docker compose`).
  3. Run keploy as that Docker image, which attaches to the app container.
     If the app isn't in Docker, keploy can't intercept its traffic on macOS.
     This is why `prepare_and_run_macos.yml` only calls `golang_docker_macos.yml`
     — there's no macOS-native equivalent.
- On **Linux** you can pick either — native (with `sudo`) or via the
  Docker image. CI exercises both (`golang_linux.yml` + `golang_docker.yml`).
- On **Windows** (amd64) native works without sudo; Docker mode also works.

**Native Linux run:**

```bash
sudo ./keploy record -c "<your app cmd>"
sudo ./keploy test   -c "<your app cmd>" --delay 10
```

If you pass a `docker`/`docker compose` command as `-c` without sudo on
Linux, `main.go` detects that in `utils.ShouldReexecWithSudo()` and
`syscall.Exec`s itself under `sudo -E` (see `utils/reexec_linux.go`). On
macOS and Windows the same helper is a no-op — `reexec_darwin.go` and
`reexec_windows.go` both short-circuit to `false`, since those platforms
rely on the active Docker context / Docker Desktop.

### Docker

```bash
sudo docker image build -t ghcr.io/keploy/keploy:v3-dev .
```

`ghcr.io/keploy/keploy:v3-dev` is the tag CI produces for dev builds and the
one the samples expect.

### Lint

Linter is `golangci-lint` with config schema v2, at `.golangci.yml`:

- Enabled linters: `govet`, `staticcheck`, `errcheck`, `ineffassign`, `unused`
- Formatters: `gofmt`, `goimports`
- Paths excluded from linters: the generated eBPF Go files (currently `pkg/agent/hooks/bpf_arm64_bpfel.go` and `pkg/agent/hooks/bpf_x86_bpfel.go`) and `pkg/service/utgen`

```bash
golangci-lint run
```

### Commit hygiene

- `.pre-commit-config.yaml` wires `commitizen` (Conventional Commits).
- `.cz.toml` pins the convention to `cz_conventional_commits`. Use types
  like `feat:`, `fix:`, `chore:`, `refactor:`, `test:`, `docs:`.
- Every commit needs a description body (blank line, then a paragraph
  explaining what changed and why).
- Sign off every commit with `git commit -s` — this appends a
  `Signed-off-by: <user.name> <user.email>` trailer using the values from
  the effective git config (system → `~/.gitconfig` → `.git/config`). Do
  not hand-construct the trailer; let git read the identity from config so
  it matches the author.

When opening PRs or issues — customer-data hygiene, PR body template, which
files not to touch — see the **`keploy-pr-workflow` skill**
(`.claude/skills/keploy-pr-workflow/SKILL.md`). Don't paste real traces,
tokens, internal hostnames, or production logs into PRs, issues, tests, or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keploy/keploy](https://github.com/keploy/keploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

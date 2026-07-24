---
trigger: always_on
description: Tyger is a REST API and CLI for **remote signal processing**. It accepts streaming
---

# Tyger — Copilot Instructions

Tyger is a REST API and CLI for **remote signal processing**. It accepts streaming
input, runs containerized user code on Kubernetes (cloud) or Docker (local), and
exposes the results via WORM data streams ("buffers"). It is deployed as an
ASP.NET Core control-plane service, a data plane service in local mode, plus
a set of Go-based CLIs and sidecars.

Always read the surrounding code before changing it, and prefer minimal,
targeted edits that match the conventions described below.

## Repository layout

| Path | Purpose |
| --- | --- |
| [server/](../server/) | .NET 10 solution ([tyger.sln](../server/tyger.sln)) |
| [server/ControlPlane/](../server/ControlPlane/) | `tyger-server` — runs jobs, manages codespecs/runs/buffers |
| [server/DataPlane/](../server/DataPlane/) | `tyger-data-plane` — local buffer I/O endpoint (Docker mode) |
| [server/Common/](../server/Common/) | Shared library (configuration, middleware, versioning, UDS) |
| [server/ControlPlane.UnitTests/](../server/ControlPlane.UnitTests/) | xUnit tests for the control plane |
| [cli/](../cli/) | Go module `github.com/microsoft/tyger/cli` (Go 1.26+) |
| [cli/cmd/tyger/](../cli/cmd/tyger/) | The `tyger` CLI entry point (cobra) |
| [cli/cmd/tyger-proxy/](../cli/cmd/tyger-proxy/) | Local HTTP/Unix-socket proxy to a remote Tyger API |
| [cli/cmd/buffer-sidecar/](../cli/cmd/buffer-sidecar/) | Sidecar that bridges named pipes <-> buffer storage |
| [cli/cmd/buffer-copier/](../cli/cmd/buffer-copier/) | Server-side cross-region buffer replication |
| [cli/cmd/worker-waiter/](../cli/cmd/worker-waiter/) | Init container for distributed runs |
| [cli/cmd/loader/](../cli/cmd/loader/) | Load/perf testing utility |
| [cli/integrationtest/](../cli/integrationtest/) | End-to-end tests (build tag `integrationtest`) |
| [cli/internal/install/](../cli/internal/install/) | `cloudinstall` (AKS/Azure) and `dockerinstall` (local) installers |
| [scripts/](../scripts/) | Shell helpers used by the Makefiles |
| [deploy/config/microsoft/](../deploy/config/microsoft/) | Source-of-truth dev/cloud/docker config templates |
| [deploy/helm/tyger/](../deploy/helm/tyger/) | Helm chart published with releases |
| [.devcontainer/](../.devcontainer/) | Dev container (Ubuntu + .NET + Go + Azure CLI + kubectl + helm) |

## Toolchain

- **.NET SDK** pinned in [server/global.json](../server/global.json) (`net10.0`,
  Nullable on, `TreatWarningsAsErrors=true`, `AnalysisMode=Recommended`,
  `RestorePackagesWithLockFile=true`). When you add or upgrade NuGet packages,
  update the corresponding `packages.lock.json` (run `make restore`).
- **Go** version is in [cli/go.mod](../cli/go.mod). Modules use a lock file
  (`go.sum`); run `go mod tidy` after dependency changes.
- **System tools** assumed on `PATH`: `make`, `bash`, `jq`, `yq`, `docker`,
  `kubectl`, `helm`, `az`, `psql`. The devcontainer installs all of these.

## Build, test, and format

The top-level [Makefile](../Makefile) is the canonical entry point for almost
every task. It selects between [Makefile.cloud](../Makefile.cloud) (default) and
[Makefile.docker](../Makefile.docker) based on the `TYGER_ENVIRONMENT_TYPE`
environment variable (`cloud` or `docker`). Some targets only exist in one of
the two; check both files before assuming a target is missing.

Common targets:

| Target | What it does |
| --- | --- |
| `make build` | `dotnet build server/tyger.sln` + `go build ./...` |
| `make build-csharp` / `make build-go` | One side only |
| `make restore` | `dotnet restore` + `go mod download` |
| `make format` | `dotnet format` (do this before pushing C# changes) |
| `make verify-format` | What CI runs; also enforces analyzer warnings |
| `make unit-test` | Runs all `*.csproj` tests and `go test ./...` (excludes the `integrationtest` build tag) |
| `make integration-test` | Brings the environment `up` and runs `cli/integrationtest` with `-tags=integrationtest` |
| `make integration-test-no-up` | Skips `up`; assumes a running deployment |
| `make integration-test-fast-only` | Adds `-fast` (skips long-running scenarios) |
| `make up` / `make down` | Install/uninstall Tyger into the target environment |
| `make install-cli` | `go install` the `tyger`, `tyger-proxy`, `buffer-sidecar` binaries with version + container image metadata baked in via `-ldflags` |
| `make run` | Run the control plane locally (after `make set-localsettings`) |
| `make full` | `make test INSTALL_CLOUD=true` (full CI-style run) |
| `make pretty-print-config-templates` | Regenerate the canonical config YAML; CI fails if the result differs |

Run the VS Code task `build` for fast incremental C# builds with the
`$msCompile` problem matcher (see [.vscode/tasks.json](../.vscode/tasks.json)).

The Go integration tests **require the `integrationtest` build tag**
(`go test -tags=integrationtest ./...` from `cli/integrationtest`). The
devcontainer's `gopls` is already configured with this tag.

## Typical inner-loop workflows

- **Changing Go code (CLI, sidecars, installers).** Edit, then
  `make install-cli` to rebuild and reinstall the `tyger`, `tyger-proxy`, and
  `buffer-sidecar` binaries on `$PATH` (with the right `-ldflags` baked in),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/tyger](https://github.com/microsoft/tyger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

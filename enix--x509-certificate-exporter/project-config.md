---
trigger: always_on
description: - `cmd/x509-certificate-exporter/` — binary entrypoint
---

# Project notes for Claude

## Layout (v4)

- `cmd/x509-certificate-exporter/` — binary entrypoint
- `pkg/cert/` — public API for certificate parsing (`pem`, `pkcs12` subpackages)
- `pkg/registry/` — public Prometheus collector + label registry
- `pkg/fileglob/` — public glob/walk engine (EXPERIMENTAL — promoted in v4 RC)
- `pkg/source/{file,k8s,kubeconfig}/` — public Source implementations (EXPERIMENTAL)
- `internal/` — wiring & process-lifecycle: config, log, server, product
- `chart/` — Helm chart

The v3 code was deleted; v4 is at the repo root. The Go module path uses `/v4`
(`github.com/enix/x509-certificate-exporter/v4/...`).

## Build & test workflow

Three tools share the workload, all wrapped by `Taskfile.yml`. Prefer
`task` targets when working interactively; `task --list` enumerates
everything.

- **Dagger Module** (`dagger.json` at repo root, source files in
  `dagger/`) — sandboxed QA/CI pipelines: lint (Go/Helm/Renovate/
  Markdown), unit tests, security scans, helm chart docs. Each
  exported method on the `X509Ce` struct is a Dagger function, called
  via `dagger call <function>` (find-up locates `dagger.json` from
  any cwd in the repo). The `New` constructor takes the working tree
  via `defaultPath="/"`, so no `--source` flag is needed.
- **GoReleaser** (`.goreleaser.yaml`) — every container image in the
  repo, dev OR release. Release pipeline: cross-compile binaries × OS/
  arch, archives, checksums, multi-arch container images for the
  scratch (default) + busybox (alt) variants, push to
  ghcr/quay/docker.io, cosign keyless on everything, GitHub Release
  (CI via `release.yaml`).
  Local: `task build:image:host` (host-arch, fast iteration),
  `task build:image:all` (every cross-arch variant). Dev: Tilt's
  `custom_build` calls `goreleaser` directly with
  `GORELEASER_TILT=1`, gating a dedicated dockers_v2 entry that uses
  `build/Dockerfile.busybox` (the alt release variant — chosen for
  dev because it ships a shell for `kubectl exec` debugging, not
  because it's the project default).
- **Direct CLI** for things that don't need a sandbox: k3d/tilt/helm
  on the dev cluster, Renovate dry-run via Docker, and `go mod tidy`
  / `go get -u` (pure toolchain operations — Dagger overhead would buy
  us nothing because `GOTOOLCHAIN=auto` makes host execution
  bit-identical anyway). GitHub Action SHA-pinning is owned by
  Renovate (`pinDigests: true` in `renovate.json5`).

| Goal | Command | Notes |
|---|---|---|
| Local binary | `task build:binary:host` | `goreleaser build --single-target --snapshot --clean` — host-arch binary under `dist/x509ce_<os>_<arch>_<v>/x509-certificate-exporter`, with a stable relative symlink at `dist/x509-certificate-exporter` (same flags / ldflags / version stamping as the release pipeline) |
| Snapshot host-arch only | `task build:image:host` | `goreleaser release --snapshot --skip=publish,sign` with `GORELEASER_LOCAL_PLATFORM=1` — fast iteration, no QEMU cross-build |
| Snapshot all images | `task build:image:all` | Like `task build:image:host` but every cross-arch variant — validates the full release matrix without pushing |
| Lint Go | `task lint:go` | `dagger call lint-go` — full golangci-lint set |
| Lint Helm | `task lint:helm` | `dagger call lint-helm` |
| Lint Markdown | `task lint:markdown` | `dagger call lint-markdown` |
| Lint all | `task lint` | Go + Helm + Renovate + Markdown |
| All tests | `task test` | runs `test:unit` + `test:fuzz` + `test:helm-examples` + `test:helm-fixtures` + `test:helm-render` + `test:e2e` sequentially |
| Unit tests | `task test:unit` | `dagger call test` — gotestsum + `-race` + coverage |
| Fuzz smoke | `task test:fuzz` | each `Fuzz*` target run for 5s — catches seed-corpus regressions |
| Helm examples | `task test:helm-examples` | `dagger call test-helm-examples` — `helm lint chart --values` on every `docs/examples/**/*.values.yaml` |
| Helm schema fixtures | `task test:helm-fixtures` | `dagger call test-helm-fixtures` — regression net for `chart/values.schema.json` (positive + paired `.expect.txt` negatives under `test/schema/{valid,invalid}/`) |
| Helm render alignment | `task test:helm-render` | `dagger call test-helm-render` — `helm template` against `test/render/all-watch-modes.yaml` and assert every ConfigMap scan path is reachable from a DaemonSet volumeMount (catches configmap↔daemonset drift) |
| End-to-end tests | `task test:e2e` | throwaway k3d cluster, Helm install, scrape `/metrics` |
| Secret leak scan | `task security:gitleaks` | `dagger call gitleaks` — open-source gitleaks CLI on the working tree; no license/token, so fork PRs scan identically |
| Vuln scan | `task security:govulncheck` | `dagger call govulncheck` |
| Vuln scan (deps) | `task security:vuln-deps` | `dagger call trivy --scan-type=fs` |
| Chart misconfig | `task security:chart-misconfig` | `dagger call trivy --scan-type=config --scan-ref=chart` |
| Tidy go.mod | `task go:tidy` | `go mod tidy` on main + `dagger/` (direct, no Dagger) |
| Bump Go deps | `task go:upgrade` | `go get -u ./...` + tidy on main + `dagger/` (direct) |
| Renovate dry-run | `task renovate:plan` | extracts deps + lists planned bumps without modifying files (debug `renovate.json5`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enix/x509-certificate-exporter](https://github.com/enix/x509-certificate-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

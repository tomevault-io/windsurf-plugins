---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Kubesplaining is a Go CLI for Kubernetes security assessment, modeled on Salesforce's Cloudsplaining. It reads cluster state (live or from a snapshot file) and emits scored findings as HTML, JSON, CSV, or SARIF.

`README.md` is the operator-facing overview. The comprehensive per-rule catalog (rule IDs, severity, detection logic, remediation, owning analyzer file) lives in [`docs/findings.md`](docs/findings.md) — read that first when adding or renaming a rule. The architectural deep-dive is in [`docs/architecture.md`](docs/architecture.md), the exclusions YAML schema in [`docs/exclusions.md`](docs/exclusions.md). The implementation roadmap and current status of every spec item is in `PLAN.md`. The full functional spec is `KUBESPLAINING_SPEC.md`.

## Common commands

Developer tooling (Go, kubectl, kind, ripgrep) is pinned via [Hermit](https://cashapp.github.io/hermit/) under `bin/`. Either activate the environment once per shell with `. ./bin/activate-hermit`, or invoke binaries directly as `./bin/go ...`, `./bin/rg ...`, etc. The Hermit shims auto-download the pinned versions on first use into `~/Library/Caches/hermit`. Docker is not Hermit-managed.

All build/test commands route through the `Makefile`, which pins `GOCACHE` / `GOMODCACHE` under `.tmp/` so module downloads stay inside the repo.

```bash
make setup           # go mod download + create bin/ and .tmp/
make build           # builds ./bin/kubesplaining from ./cmd/kubesplaining
make test            # go test ./...
make lint            # gofmt -l check + go vet ./...
make e2e             # boots a kind cluster, applies testdata/e2e/vulnerable.yaml, runs the CLI, asserts expected rule IDs
make clean           # removes ./bin, ./kubesplaining-report, ./.tmp
make install-hooks   # activate .githooks/ pre-commit + commit-msg hooks (one-time per clone)
```

Commits follow [Conventional Commits](https://www.conventionalcommits.org/) — the `commit-msg` hook enforces it once `make install-hooks` has run. The `pre-commit` hook runs `gofmt -l` and `golangci-lint` over the packages of staged `.go` files; install the linter once with `./bin/hermit install golangci-lint`. See `.githooks/README.md` for details and bypass.

Single-package or single-test runs (use the same `GOCACHE` / `GOMODCACHE` env so you do not redownload modules):

```bash
GOCACHE=$(pwd)/.tmp/go-build-cache GOMODCACHE=$(pwd)/.tmp/go-mod-cache \
  go test ./internal/analyzer/rbac/...

GOCACHE=$(pwd)/.tmp/go-build-cache GOMODCACHE=$(pwd)/.tmp/go-mod-cache \
  go test ./internal/analyzer/privesc -run TestFindPaths -v
```

`make e2e` requires a reachable Docker daemon (see `scripts/kind-e2e.sh`); `kind`, `kubectl`, and `rg` come from Hermit once the environment is activated. The script greps `findings.json` for specific rule IDs — when you add or rename a rule that the e2e fixture should produce, update the `rg -q "KUBE-..."` assertions and/or `testdata/e2e/vulnerable.yaml` together.

`gofmt -l` is the lint gate; the Makefile lists Go files via `rg --files -g '*.go'` (Hermit-managed).

## Architecture

Four-stage pipeline: **connection → collection → analysis → report**. The boundary that matters most: the **collector is the only thing that talks to the Kubernetes API**; analyzers consume `models.Snapshot` and never make network calls. This is what enables `download` → `scan --input-file` for offline analysis.

```
cmd/kubesplaining/main.go            # entrypoint, ldflags-injected version
└── internal/cli/                     # cobra commands: download, scan, scan-resource, report, create-exclusions-file, version
    └── internal/connection/          # client-go credentials resolution
    └── internal/collector/           # parallel API listing → models.Snapshot (single ~657-line collector.go)
    └── internal/manifest/            # offline alternative to collector: reads a YAML/JSON manifest into a Snapshot for `scan-resource`
    └── internal/analyzer/            # the engine + 7 modules; see below
    └── internal/exclusions/          # YAML-driven post-analysis muting; matched findings are dropped from output (see docs/exclusions.md for schema)
    └── internal/report/              # html/json/csv/sarif writers; HTML rendering is split across summary.go, evidence_render.go, attack_graph.go, glossary.go (each 400-850 lines because all CSS/JS is embedded)
    └── internal/scoring/             # composite formula + clamp + threshold helper, shared by analyzers and engine
    └── internal/permissions/         # aggregate.go: collapses (Cluster)RoleBindings × (Cluster)Roles into per-subject EffectiveRules
    └── internal/models/              # Snapshot, Finding, Severity, EscalationGraph/Path/Hop — the cross-package vocabulary
```

### The analyzer engine (`internal/analyzer/engine.go`)

Every analyzer module implements:

```go
type Module interface {
    Name() string
    Analyze(ctx context.Context, snapshot models.Snapshot) ([]models.Finding, error)
}
```

`Engine.Analyze` runs all selected modules **in parallel goroutines**, then post-processes:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0hardik1/kubesplaining](https://github.com/0hardik1/kubesplaining) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

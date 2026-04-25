---
trigger: always_on
description: > Lightweight map for autonomous coding agents. Read this first, then navigate to specific docs on demand.
---

# gcx — Agent & Developer Entry Point

> Lightweight map for autonomous coding agents. Read this first, then navigate to specific docs on demand.

## Quick Start

**gcx** is a unified CLI for managing Grafana resources. It operates in two tiers: (1) a **K8s resource tier** that uses Grafana 12+'s Kubernetes-compatible API via `k8s.io/client-go` for dashboards, folders, and other K8s-native resources, and (2) a **Cloud provider tier** with pluggable providers for Grafana Cloud products (SLO, Synthetic Monitoring, IRM, Fleet Management, etc.) that use product-specific REST APIs. Built in Go, it uses Cobra for CLI structure.

## Documentation Map

| File | Purpose |
|------|---------|
| [VISION.md](VISION.md) | Goals, product surface, roadmap themes, release timeline |
| [CONSTITUTION.md](CONSTITUTION.md) | Invariants — things that cannot change without explicit human approval |
| [ARCHITECTURE.md](ARCHITECTURE.md) | System overview (all 7 subsystems), pipeline diagrams, ADR index |
| [DESIGN.md](DESIGN.md) | CLI UX design: command grammar, output model, exit codes |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Dev setup, testing environment, contribution workflow |
| [docs/architecture/](docs/architecture/) | Deep-dive architecture docs (patterns, resource model, CLI layer, data flows, …) |
| [docs/design/](docs/design/) | Prescriptive UX implementation rules (output, errors, agent mode, naming, …) |
| [docs/reference/](docs/reference/) | Provider guides, CLI reference, migration analysis |
| [docs/_templates/](docs/_templates/) | Spec and planning templates (feature, bugfix, refactor, ADR, research) |

## Architecture at a Glance

Two tiers: **K8s resource tier** (dashboards, folders via `/apis`) and **Cloud provider tier** (SLO, SM, IRM, etc. via product REST APIs). See [ARCHITECTURE.md](ARCHITECTURE.md) for pipeline diagrams and extension pipelines.

## Key Conventions

> Authoritative source: [CONSTITUTION.md](CONSTITUTION.md) (invariants) and [DESIGN.md](DESIGN.md) (UX rules). This is the quick-reference summary.

- **Options pattern**: Every command uses `opts struct` + `setup(flags)` + `Validate()` + constructor
- **Processor pipeline**: `Processor.Process(*Resource) error` — composable transformations for push/pull
- **errgroup concurrency**: Bounded parallelism (default 10) for all batch I/O operations
- **Folder-before-dashboard**: Push pipeline does topological sort — folders pushed level-by-level before other resources
- **Config = kubectl kubeconfig**: Named contexts with server/auth/namespace, env var overrides
- **Format-agnostic data fetching**: Commands fetch all data regardless of `--output` format; codecs control display, not data acquisition (see Pattern 13 in `docs/architecture/patterns.md`)
- **PromQL via promql-builder**: Use `github.com/grafana/promql-builder/go/promql` for PromQL construction, not string formatting (see Pattern 14 in `docs/architecture/patterns.md`)
- **Portable agent skills live under `claude-plugin/skills/`**: Treat that tree as the canonical portable Agent Skills bundle. Do not add distributable gcx skills under repo-local `.agents/skills/` — that changes repo-context discovery semantics for tools that scan `.agents`.

## Essential Commands

```bash
make build       # Build to bin/gcx
make tests       # Run all tests with race detection
make lint        # Run golangci-lint
make all         # lint + tests + build + docs
make docs        # Generate + build all documentation
```

**Without devbox**: replace `make` targets with direct Go commands — `go build -buildvcs=false -o bin/gcx ./cmd/gcx/` and `go test ./...`. Always build to `bin/gcx`.

> **Agent environments**: always prefix with `GCX_AGENT_MODE=false` — agent-mode auto-detection changes output defaults in `make docs`, producing wrong CLI reference docs.

## Testing

```bash
go test ./internal/providers/traces/...   # Run one package
go test -run TestQueryCodec ./internal/... # Run matching tests across packages
go test -race -count=1 ./...              # Full suite with race detection (same as make tests)
```

Prefer table-driven tests. See existing `_test.go` files for patterns.

## Package Map

> Full map with sub-packages: [docs/architecture/project-structure.md](docs/architecture/project-structure.md)

```
cmd/gcx/
  root/         CLI root (logging, global flags)
  auth/         OAuth login (browser PKCE)
  config/       Config management (set, use-context, view)
  resources/    Resource commands (get, schemas, push, pull, delete, edit, validate)
  dashboards/   Dashboard snapshot (Image Renderer)
  datasources/  Datasource commands (list, get, query, per-type subcommands via DatasourceProvider)
  providers/    Provider list command
  assistant/    Assistant commands (AI-powered investigations)
  api/          Raw API passthrough
  linter/       Linting (mounted under dev lint)
  commands/     Commands catalog (agent metadata)
  helptree/     Help tree for agent context
  setup/        Onboarding + instrumentation
  skills/       Portable Agent Skills installer for .agents-compatible tools
  dev/          Developer tools (import, scaffold, generate, lint, serve)
  fail/         Structured error conversion

internal/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/gcx](https://github.com/grafana/gcx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

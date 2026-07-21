---
trigger: always_on
description: **Kubebuilder** is a **framework** and **command-line interface (CLI)** for building **Kubernetes APIs** using **Custom Resource Definitions (CRDs)**.
---

# Kubebuilder AI Agent Guide

**Kubebuilder** is a **framework** and **command-line interface (CLI)** for building **Kubernetes APIs** using **Custom Resource Definitions (CRDs)**.
It provides scaffolding and abstractions that accelerate the development of **controllers**, **webhooks**, and **APIs** written in **Go**.

## Quick Reference

| Item       | Value                                                     |
|------------|-----------------------------------------------------------|
| Language   | Defined in the go.mod                                     |
| Module     | `sigs.k8s.io/kubebuilder/v4`                              |
| Binary     | `./bin/kubebuilder`                                       |
| Core deps  | `controller-runtime`, `controller-tools`, Helm, Kustomize |
| Docs       | https://book.kubebuilder.io                               |


## Directory Map

```
pkg/
  cli/              CLI command implementations
    alpha/          Alpha/experimental commands (generate, update, etc.)
    init.go         'init' command + default PluginBundle definition
    api.go          'create api' command
    webhook.go      'create webhook' command
    edit.go         'edit' command
    root.go         Root command setup
  machinery/        Scaffolding engine (templates, markers, injectors)
    template.go     Base template interface
    inserter.go     Code injection engine
    marker.go       Marker detection and processing
    filesystem.go   Filesystem abstraction (uses afero)
  model/
    resource/       Resource model (GVK, API, Controller, Webhook)
    stage/          Plugin execution stages
  plugin/           Plugin interfaces and utilities
    interface.go    Core plugin interfaces (Plugin, Init, CreateAPI, etc.)
    bundle.go       Plugin composition
    util/           Helper functions for plugin authors
  plugins/          Plugin implementations (ADD NEW PLUGINS HERE)
    golang/v4/      Main Go scaffolding (default for go projects)
      scaffolds/    Scaffolding for init, api, webhook
        internal/templates/  Template implementations
    golang/deployimage/  Deploy-image pattern plugin
    common/kustomize/v2/  Kustomize manifest generation (default)
    optional/       Optional plugins (enabled via --plugins flag)
      helm/         Helm chart generation (v1alpha deprecated, v2alpha current)
      grafana/      Grafana dashboard generation
      autoupdate/   Auto-update GitHub workflow
    external/       External plugin support (exec-based plugins)
docs/book/          mdBook documentation (https://book.kubebuilder.io)
  src/              Markdown source files
    **/testdata/    Sample projects used in docs (regenerated)
test/
  e2e/              E2E tests requiring Kubernetes cluster
    v4/             Tests for v4 plugin
    helm/           Tests for Helm plugin
    deployimage/    Tests for deploy-image plugin
    utils/          Test helpers (TestContext, etc.)
  testdata/         Scripts to generate testdata projects
    generate.sh     Main generation script
    test.sh         Tests all testdata projects
testdata/           Generated complete sample projects (DO NOT EDIT)
  project-v4/                    Basic v4 project
  project-v4-multigroup/         Multigroup project
  project-v4-with-plugins/       Project with optional plugins
hack/docs/          Documentation generation
  generate.sh       Regenerate docs samples + marker docs
  generate_samples.go  Sample generation logic
cmd/                CLI entry point
  version.go        Version info (updated by make update-k8s-version)
main.go             Application entry point
```

**Key Locations for Common Tasks:**
- Add new plugin → `pkg/plugins/<category>/<name>/`
- Add new template → `pkg/plugins/<plugin>/scaffolds/internal/templates/`
- Modify CLI commands → `pkg/cli/`
- Add scaffolding machinery → `pkg/machinery/`
- Add tests → `test/e2e/all/plugin_<name>_test.go` or `pkg/<package>/*_test.go`

## Critical Rules

### Do Not Manually Edit Generated Files
- `testdata/` - regenerated via `make generate-testdata`
- `docs/book/**/testdata/` - regenerated via `make generate-docs`
- `*/dist/chart/` - regenerated via `make generate-charts`

### File-Specific Requirements

After making changes, run the appropriate commands based on what you modified:

**Generate Commands (rebuild artifacts):**
- **If you modify files in `hack/docs/internal/`** → run `make install && make generate-docs`
- **If you modify files in `pkg/plugins/optional/helm/`** → run `make install && make generate-charts`
- **If you modify any boilerplate/template files** → run `make install && make generate`

**Formatting Commands:**
- After editing `*.go` → `make lint-fix`
- After editing `*.md` → `make remove-spaces`

**Always Run Before PR:**
```bash
make lint-fix    # Auto-fix Go code style
make test-unit   # Verify unit tests pass
```

**Note:** Boilerplate/template files are Go files that define scaffolding templates, typically located in `pkg/plugins/**/scaffolds/internal/templates/` or files that generate code/configs for scaffolded projects.

## Development Workflow

### Build & Install
```bash
make build    # Build to ./bin/kubebuilder

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubernetes-sigs/kubebuilder](https://github.com/kubernetes-sigs/kubebuilder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

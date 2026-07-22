---
trigger: always_on
description: This document is for AI agents working in the OpenShift Installer codebase. It supplements the existing docs -- read `CLAUDE.md` for build/test commands, `README.md` for user-facing overview, and `CONTRIBUTING.md` for contribution workflow.
---

# AGENTS.md

This document is for AI agents working in the OpenShift Installer codebase. It supplements the existing docs -- read `CLAUDE.md` for build/test commands, `README.md` for user-facing overview, and `CONTRIBUTING.md` for contribution workflow.

## Guideline Documents

These files contain detailed, domain-specific guidance. Read the relevant one before making changes in that area:

- [`docs/security-guidelines.md`](docs/security-guidelines.md) -- TLS, credentials, FIPS compliance, secrets handling
- [`docs/error-handling-guidelines.md`](docs/error-handling-guidelines.md) -- Error wrapping, validation patterns, cloud error handling
- [`docs/testing-guidelines.md`](docs/testing-guidelines.md) -- Mocks, table-driven tests, integration test setup
- [`docs/integration-guidelines.md`](docs/integration-guidelines.md) -- Cloud platform structure, Cluster API, tagging, SDK usage
- [`docs/performance-guidelines.md`](docs/performance-guidelines.md) -- Concurrency, rate limiting, destroy flow optimization

## Code Style and Conventions

### Import Organization

Imports are sorted into four groups by `hack/go-fmt.sh` using the [gci tool](https://github.com/daixiang0/gci):

1. Standard library
2. Third-party packages
3. `github.com/openshift` packages (project-internal prefix)
4. Blank imports (for side effects)

Groups are separated by blank lines. Always run `hack/go-fmt.sh .` before submitting.

### Import Alias Conventions

Platform-specific imports frequently collide on package names. The codebase uses consistent alias prefixes:

- Type packages: `awstypes`, `azuretypes`, `gcptypes`, `vspheretypes`, etc.
- Install-config sub-assets: `icazure`, `icgcp`, `icibmcloud`, etc.
- Infrastructure CAPI providers: `awscapi`, `gcpcapi`, `vspherecapi`, etc.
- Infrastructure packages: `azureinfra`, `baremetalinfra`, etc.
- CAPI manifests: `capimanifests`

When adding a new import that conflicts with an existing package name, follow the alias pattern already used in that file or the patterns above.

### Package Documentation

Packages in `pkg/types/` should have a `doc.go` file with a package comment and `+k8s:deepcopy-gen=package` directive. Platform-type packages also define their platform `Name` constant in `doc.go`:

```go
// Package aws contains AWS-specific structures for installer
// configuration and management.
// +k8s:deepcopy-gen=package
package aws

// Name is name for the AWS platform.
const Name string = "aws"
```

### Generated Code

Several files are generated and must not be edited by hand:

- `zz_generated.deepcopy.go` -- deep copy methods for types, generated via `+k8s:deepcopy-gen=package` directive
- `data/data/install.openshift.io_installconfigs.yaml` -- CRD definition, regenerated with `go generate ./pkg/types/installconfig.go`
- Mock files under `pkg/asset/mock/` -- regenerated with `hack/go-genmock.sh`

If you change an interface that has mocks or a type that has deepcopy, you must regenerate.

## Go Module and Vendoring

- The module is `github.com/openshift/installer`.
- All dependencies are vendored in `vendor/`. The `vendor/` directory is checked in.
- **Critical rule**: Always commit vendored dependency changes in a separate commit from functional code changes. This is enforced by convention and makes reviews tractable.
- When updating dependencies: `go get <pkg>`, then `go mod tidy`, then `go mod vendor`.
- Updating `github.com/openshift/api` is a special case that also requires regenerating the install-config CRD (see `CLAUDE.md`).

## Architecture: What Is Not Obvious

### The Asset DAG

The installer's core architecture is a directed acyclic graph (DAG) of "assets." Full design is in `docs/design/assetgeneration.md`, but here are the essentials for working in the code:

**Every piece of installer output is an Asset.** The `Asset` interface (`pkg/asset/asset.go`) has three methods:
- `Dependencies() []Asset` -- declares what this asset needs
- `Generate(ctx, Parents) error` -- produces the asset from its dependencies
- `Name() string` -- human-readable identifier

**`WritableAsset`** extends `Asset` with `Files()` and `Load()` -- it can be serialized to disk and read back. The installer chains targets: `install-config` -> `manifests` -> `ignition-configs` -> `cluster`. Each target consumes (and removes from disk) the previous target's files.

**Targets** are defined in `pkg/asset/targets/targets.go`. They group the writable assets for each CLI command (`create install-config`, `create manifests`, `create ignition-configs`, `create cluster`).

**The Store** (`pkg/asset/store/`) manages the DAG resolution. It does depth-first traversal, generating dependencies before dependents. Assets can be loaded from disk (user-provided overrides) or from an internal state file.

**When adding a new asset:**
1. Create a struct implementing `Asset` (or `WritableAsset` if it produces files)
2. Declare dependencies in `Dependencies()`
3. In `Generate()`, call `parents.Get(...)` to retrieve dependency state
4. If writable, add it to the appropriate target list in `pkg/asset/targets/targets.go`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/installer](https://github.com/openshift/installer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

---
trigger: always_on
description: Guidance for agentic coding agents working in this repository.
---

# AGENTS.md

Guidance for agentic coding agents working in this repository.

## Project Overview

OpenShift Preflight is a Go CLI tool for Red Hat Software Certification that validates partner-submitted containers and operator bundles. Module: `github.com/redhat-openshift-ecosystem/openshift-preflight`.

## Build Commands

```bash
make build                    # Build the preflight binary
make build-multi-arch-linux   # Build for amd64, arm64, ppc64le, s390x
make build-multi-arch-mac     # Build for amd64, arm64 (Darwin)
```

All builds use `CGO_ENABLED=0` and `-trimpath`. Version metadata is injected via `-ldflags`.

## Test Commands

```bash
make test          # Run all unit tests (excludes e2e)
make cover         # Run tests with -race and coverage report
make test-e2e      # End-to-end tests (requires live OpenShift cluster)
```

The `-tags testing` build tag is **required** for all unit test runs.

### Running a Single Package's Tests

```bash
go test -v -tags testing \
  -ldflags "-X github.com/redhat-openshift-ecosystem/openshift-preflight/version.commit=bar \
            -X github.com/redhat-openshift-ecosystem/openshift-preflight/version.version=foo" \
  ./internal/policy/container/
```

### Running a Single Ginkgo Spec or Describe Block

Use `--ginkgo.focus` to filter by description string. Run from the package directory:

```bash
go test -v -tags testing \
  -ldflags "-X github.com/redhat-openshift-ecosystem/openshift-preflight/version.commit=bar \
            -X github.com/redhat-openshift-ecosystem/openshift-preflight/version.version=foo" \
  -run TestContainer \
  --ginkgo.focus="HasLicense" \
  ./internal/policy/container/
```

The `-run` value must match the `TestXxx` suite function name defined in the `*_suite_test.go` file for that package. Common mappings:

| `-run` flag            | Package path                        |
|------------------------|-------------------------------------|
| `TestInternalEngine`   | `./internal/engine/`                |
| `TestContainer`        | `./internal/policy/container/`      |
| `TestOperator`         | `./internal/policy/operator/`       |
| `TestRuntime`          | `./internal/runtime/`               |

## Code Quality Commands

```bash
make lint   # golangci-lint run --build-tags testing
make fmt    # gofumpt -l -w . && golangci-lint fmt (must produce no diff)
make vet    # go vet -tags testing ./...
make tidy   # go mod tidy (must produce no diff)
```

All PRs must pass lint, fmt, vet, and tidy with no diffs.

## Code Style

### Formatting

- Formatters: **gofumpt** (stricter than `gofmt`) and **golangci-lint fmt**. Run `make fmt` before committing.
- `make fmt` runs both `gofumpt -l -w .` and `golangci-lint fmt` to ensure consistent formatting and import organization.
- Do not manually format; let the tooling handle it.

### Import Organization

Three groups, separated by blank lines, enforced by `golangci-lint` (via the `goimports` linter with local prefix):

```go
import (
    // 1. Standard library
    "context"
    "fmt"

    // 2. Third-party dependencies
    "github.com/go-logr/logr"
    "github.com/google/go-containerregistry/pkg/crane"

    // 3. Project-local (github.com/redhat-openshift-ecosystem/openshift-preflight/...)
    "github.com/redhat-openshift-ecosystem/openshift-preflight/internal/check"
    "github.com/redhat-openshift-ecosystem/openshift-preflight/internal/image"
)
```

### Required Import Aliases

The `importas` linter enforces these aliases — use them exactly:

| Import path                                                      | Alias               |
|------------------------------------------------------------------|---------------------|
| `k8s.io/api/core/v1`                                             | `corev1`            |
| `k8s.io/apimachinery/pkg/apis/meta/v1`                           | `metav1`            |
| `k8s.io/apimachinery/pkg/api/errors`                             | `apierrors`         |
| `github.com/operator-framework/api/pkg/operators/v1alpha1`       | `operatorsv1alpha1` |
| `github.com/operator-framework/api/pkg/operators/v1`             | `operatorsv1`       |
| `github.com/openshift/api/image/v1`                              | `imagestreamv1`     |

Ginkgo and Gomega use dot imports in test files (allowed by lint config):
```go
import (
    . "github.com/onsi/ginkgo/v2"
    . "github.com/onsi/gomega"
)
```

### Naming Conventions

- Types and exported symbols: `PascalCase`
- Unexported implementation types: `camelCase` (e.g., `craneEngine`)
- Interfaces are exported; implementations may be unexported
- Check structs: name matches the `Name()` return (e.g., `HasLicenseCheck` → `"HasLicense"`)
- Constructors: `New<Type>(...)` returning the interface or concrete type
- Method receivers: short abbreviation of the receiver type (e.g., `p *HasLicenseCheck`, `c *craneEngine`)
- Package-level sentinel errors: `var errSomething = errors.New("...")`

### Error Handling

- Wrap errors with `fmt.Errorf("context: %w", err)` to preserve the chain
- Use `%v` for non-wrapped formatting, `%w` when callers may use `errors.Is`/`errors.As`
- Use `errors.Is(err, target)` for sentinel comparisons — never compare error strings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redhat-openshift-ecosystem/openshift-preflight](https://github.com/redhat-openshift-ecosystem/openshift-preflight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

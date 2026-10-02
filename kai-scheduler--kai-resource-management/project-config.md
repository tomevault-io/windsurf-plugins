---
trigger: always_on
description: KAI Resource Management is an open source, Kubernetes-native resource
---

# KAI Resource Management — Agent Development Guide

KAI Resource Management is an open source, Kubernetes-native resource
management layer for KAI Scheduler. It provides the organizational and
infrastructure abstractions around scheduling, including projects,
departments, node pools, queues, and workload placement.

These instructions apply to the entire repository.

When KAI Scheduler already has a repository pattern for Makefiles, directories,
workflows, or development tooling, mirror it closely. Do not introduce a new
abstraction or structure unless this repository requires it or the user approves
the divergence.

## Supported commands

Use the root Makefile as the public development interface:

```bash
make help             # List supported targets
make fmt-go           # Format Go files
make vet-go           # Run go vet
make lint-go          # Run the pinned golangci-lint version
make lint-go-host     # Same linter, host toolchain: no Docker
make lint             # Run formatting and static checks
make test-chart       # Run chart unit tests in the pinned container
make test             # Run non-e2e Go and Helm tests
make validate         # Run all non-mutating repository validation except tests
make gen-license      # Add missing Apache-2.0 source headers
```

For a behavior-changing pull request, create a changelog fragment:

```bash
make changelog KIND=Added BODY="Add configurable project namespace prefix"
```

Valid changelog kinds are `Added`, `Changed`, `Fixed`, and `Removed`. Keep the
body user-facing and under 20 words.

Never edit `CHANGELOG.md` directly; it is written at release time. Pull requests
that do not change behavior apply the `skip-changelog` label instead of adding a
fragment; dependency updates use `dependencies`.

## Repository structure

The repository deliberately uses one Go module and one Makefile.

- `cmd/<name>/` — executable entry points and process wiring.
- `pkg/<name>/` — shared Go implementation.
- `deployments/<name>/` — deployment configuration and Helm charts. The
  primary chart belongs at `deployments/kai-resource-management-chart/`.
- `docs/` — user, administrator, reference, and developer documentation.
- `hack/` — maintenance, generation, and developer integration scripts.
- `test/e2e/` — end-to-end suites and framework when their separate
  infrastructure is introduced.
- `.agents/` — repository-owned skills and other shared agent assets.

Do not add:

- Nested `go.mod` or `go.work` files.
- Component-specific Makefiles.
- A top-level `examples` directory.
- Generated build output or downloaded Helm dependencies.

Examples and sample manifests belong beside their documentation under `docs/`.

## Go conventions

### Package and command design

- Keep `main` packages small. Configuration parsing and process wiring belong
  in `cmd`; testable behavior belongs in `pkg`.
- Keep packages cohesive and domain-named. Avoid generic `utils`, `helpers`, or
  `common` packages unless the domain itself is genuinely common.
- Prefer the standard library before introducing dependencies.
- Accept `context.Context` as the first parameter for operations that perform
  I/O, block, or may be canceled.
- Return errors to the caller. Do not panic for expected runtime failures.
- Wrap errors with actionable context using `%w`.
- Avoid mutable package globals.

### Imports

Organize imports into three groups separated by blank lines:

```go
import (
    "context"
    "fmt"

    corev1 "k8s.io/api/core/v1"
    "sigs.k8s.io/controller-runtime/pkg/client"

    "github.com/kai-scheduler/kai-resource-management/pkg/example"
)
```

The groups are standard library, external dependencies, and repository
packages.

### Naming

- Go files use `snake_case.go`.
- Exported identifiers use PascalCase; unexported identifiers use camelCase.
- Boolean functions use an `Is`, `Has`, `Can`, or `Should` prefix where it
  improves clarity.
- Interfaces describe behavior and normally use an `-er` name.
- Avoid abbreviations unless they are standard Kubernetes or domain terms.

### Logging

- Use structured logging and stable field names.
- Obtain request or reconciliation loggers from context when supported.
- Include object kind, namespace, and name where relevant.
- Do not log credentials, tokens, certificates, complete Secrets, or
  personally identifiable information.
- Use debug verbosity for high-volume diagnostic events.

### Comments and headers

- `make gen-license` is authoritative for headers on hand-written source and
  configuration files and emits language-native line comments.
- `hack/boilerplate.go.txt` and `hack/boilerplate.yaml.txt` provide the same
  headers to source generators such as `controller-gen`.
- Comment an exported Go declaration only when the comment says something the
  signature does not. Never restate the function name, its parameters or its
  return type: `// Scheme returns the scheme` on `func (c *Controller) Scheme()
  *runtime.Scheme` is noise, and so is describing a parameter that is already
  visible in the signature. Prefer no comment to an obvious one.
- Comments explain why a choice or invariant exists, not what obvious code
  does.
- Preserve upstream headers on generated files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kai-scheduler/kai-resource-management](https://github.com/kai-scheduler/kai-resource-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->

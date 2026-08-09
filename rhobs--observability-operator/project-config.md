---
trigger: always_on
description: Guidelines for AI coding assistants working on this project. This file
---

# Agent Guidelines — Observability Operator

Guidelines for AI coding assistants working on this project. This file
deliberately avoids information that ages (versions, directory listings, image
tags). Consult the authoritative sources referenced below instead.

## Quick reference

| What                  | Where / command                                      |
|-----------------------|------------------------------------------------------|
| Go version            | `go.mod` (`go` directive)                            |
| Build                 | `make operator`                                      |
| Unit tests            | `make test-unit`                                     |
| E2E tests             | `make test-e2e` (requires a running cluster)         |
| Lint                  | `make lint` (Go + shell)                             |
| Code generation       | `make generate`                                      |
| Install dev tools     | `make tools` (installs to `tmp/bin/`)                |
| Container runtime     | `podman` preferred, `docker` fallback                |
| All Makefile targets  | `make help` or read `Makefile` and `Makefile.tools`  |

## Project overview

The Cluster Observability Operator (COO) is a Kubernetes operator for
OpenShift that manages observability components. It is built with
controller-runtime and distributed via OLM.

Key custom resources (all `v1alpha1`):

- **MonitoringStack** (`monitoring.rhobs`) — deploys Prometheus,
  Alertmanager, and Thanos sidecar.
- **ThanosQuerier** (`monitoring.rhobs`) — federates queries across
  MonitoringStacks.
- **UIPlugin** (`observability.openshift.io`, cluster-scoped) — deploys
  OpenShift Console dynamic plugins (dashboards, logging, tracing,
  troubleshooting panel, monitoring).
- **ObservabilityInstaller** (`observability.openshift.io`) — installs
  Tempo and OpenTelemetry operators via OLM subscriptions.

API types live in `pkg/apis/` which is a **separate Go module**
(`pkg/apis/go.mod`) so downstream consumers can import types without the
full operator dependency tree.

## Forked prometheus-operator

The operator depends on `github.com/rhobs/obo-prometheus-operator`, a
Red Hat fork of prometheus-operator. CRDs from this fork use the
`monitoring.rhobs` API group (not upstream `monitoring.coreos.com`).
When working with Prometheus, Alertmanager, or ServiceMonitor types,
import from `github.com/rhobs/obo-prometheus-operator/pkg/apis/monitoring/v1`.

## Code conventions

### Imports

Imports are grouped and ordered by `gci` (enforced via `.golangci.yml`):

1. Standard library
2. Third-party
3. `github.com/rhobs/observability-operator` (project-local)

### Error handling

Either log the error **or** return it — never both. This was explicitly
cleaned up in PR #1148. The project treats "log and return" as a bug.

### Reconciliation pattern

All controllers use **server-side apply** via controller-runtime's
`client.Apply`:

```go
client.Patch(ctx, obj, client.Apply, client.ForceOwnership, client.FieldOwner("observability-operator"))
```

Managed resources carry the label
`app.kubernetes.io/managed-by: observability-operator`.

The generic reconciler abstraction lives in `pkg/reconciler/`. Use the
existing `Updater`, `Deleter`, and `CreateOrUpdater` types rather than
hand-rolling owner reference management.

### RBAC

RBAC is generated from `+kubebuilder:rbac` markers on controller files.
After modifying markers, run `make generate` to regenerate
`deploy/operator/observability-operator-cluster-role.yaml`.

### OpenShift feature gate

Functionality that requires OpenShift APIs (UIPlugins,
ObservabilityInstaller, operator self-monitoring, TLS via service CA) is
gated behind the `--openshift.enabled` CLI flag. The corresponding
struct path is `FeatureGates.OpenShift.Enabled` in
`pkg/operator/operator.go`. When adding OpenShift-only controllers or
features, gate them the same way.

### UIPlugin compatibility matrix

UIPlugin images and support levels are managed through the compatibility
matrix in `pkg/controllers/uiplugin/compatibility_matrix.go`. When
adding or updating a console plugin, update the matrix entries. Tests
for the matrix are in `compatibility_matrix_test.go`.

## Commit conventions

Commits follow [Conventional Commits](https://www.conventionalcommits.org/)
enforced by `commitlint` (config: `commitlint.config.mjs`). CI rejects
non-conforming messages.

Common prefixes observed in this project:

| Prefix    | Use for                                     |
|-----------|---------------------------------------------|
| `feat:`   | New features or capabilities                |
| `fix:`    | Bug fixes                                   |
| `chore:`  | Maintenance, dep bumps, refactors           |
| `test:`   | Test-only changes                           |
| `docs:`   | Documentation                               |
| `build:`  | Build system, CI                            |
| `api:`    | API type changes                            |

PR titles often carry a Jira ticket prefix, e.g.
`COO-1234: fix: description` or `NO-JIRA: chore: description`.
Use `NO-JIRA:` or `NO-ISSUE:` when there is no associated ticket.

Sign commits: `git commit -s`.

## Testing

### Unit tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhobs/observability-operator](https://github.com/rhobs/observability-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

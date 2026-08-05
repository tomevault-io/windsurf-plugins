---
trigger: always_on
description: This file contains the **unified rules and conventions** that every AI agent working on the KubeKey v4 codebase must follow.
---

# KubeKey Agent Guide

This file contains the **unified rules and conventions** that every AI agent working on the KubeKey v4 codebase must follow.

It is intentionally **not** a project tour or per-role workflow. For project architecture and code logic details, see:

- [README.md](README.md) – user-facing intro.
- [docs/en/framework/README.md](docs/en/framework/README.md) – writing custom playbooks.
- [.opencode/CODE_LOGIC.md](.opencode/CODE_LOGIC.md) – detailed code logic flows.

For per-role instructions, see `.opencode/agents/`:

- [maintainer.md](.opencode/agents/maintainer.md) – default entry point (orchestrator); drives the full Agent pipeline for end-to-end requests.
- [architect.md](.opencode/agents/architect.md) – analyze requirements and produce design documents.
- [developer.md](.opencode/agents/developer.md) – implement code and update docs based on the design document.
- [reviewer.md](.opencode/agents/reviewer.md) – review code and produce review/PR artifacts.
- [tester.md](.opencode/agents/tester.md) – plan and execute tests.

## 1. What is KubeKey?

KubeKey v4 is a Go-based task execution framework modeled on Ansible. Its primary use case is installing and managing Kubernetes clusters, but the core engine is generic: it loads playbook projects (YAML), executes tasks across hosts via connectors (SSH/local/Kubernetes/Prometheus), and provides built-in modules (command, copy, template, image, etc.).

Two binaries are produced:

- `kk` – CLI tool that runs playbooks locally or inside a Kubernetes pod.
- `kk-controller-manager` – Kubernetes operator that watches `Playbook` CRs and spawns executor pods.

## 2. Repository Layout

```text
/Users/liujian/code/kubesphere/kubekey
├── cmd/kk                    # CLI binary entry
├── cmd/controller-manager    # Operator binary entry
├── api/                      # Separate Go module for CRD Go types
├── pkg/                      # Core packages
│   ├── executor/             # Playbook/role/block/task execution engine
│   ├── project/              # Project loading (builtin/local/git)
│   ├── modules/              # Built-in modules
│   ├── variable/             # Variable merging and lookup
│   ├── connector/            # SSH/local/k8s/prometheus connectors
│   ├── converter/            # Block↔Task conversion, template rendering
│   ├── manager/              # commandManager/controllerManager/webManager
│   ├── controllers/          # Kubernetes reconcilers and webhooks
│   ├── proxy/                # Hybrid REST API proxy
│   ├── web/                  # HTTP services
│   ├── const/                # Constants, scheme, workdir helpers
│   └── utils/                # Small utilities
├── builtin/core/             # Embedded playbooks/roles (requires "builtin" tag)
├── plugins/                  # Optional community playbooks/roles
├── config/                   # Generated CRDs, Helm charts, Kustomize
├── docs/                     # Documentation
│   └── en/framework/         # User-facing framework docs
├── Makefile                  # Build targets, generate, test, lint
├── go.mod                    # Main module
├── go.work                   # Workspace including ./api
├── version/                  # Build-time version injection
├── .opencode/
│   ├── agents/               # Agent role definitions
│   └── CODE_LOGIC.md         # Shared code logic reference for all agents
└── _output/agents/           # Agent-generated intermediate artifacts
```

## 3. Universal Conventions

All agents must follow these conventions when producing or modifying code.

### 3.1 Logging

Choose the appropriate log level.

| Level | Usage |
|-------|-------|
| `klog.Info` | Main business events. |
| `klog.Warning` | Recoverable abnormal situations. |
| `klog.Error` | Errors requiring attention. |
| `klog.V(4)` | Framework execution flow. Examples: `project`, `proxy`, `variable`, `connector`, `web`, `manager`, `controllers`, `executor`. |
| `klog.V(5)` | Extension modules. Examples: `module`, `converter`. |
| `klog.V(6)` | Debug information. May include detailed intermediate values and execution flow. |

### 3.2 Errors

Wrap errors only where they originate.

- Lower layers should use `errors.Wrap` (or equivalent) to add context.
- Upper layers should return the error directly unless adding meaningful business context.
- Do not repeatedly wrap the same error.

KubeKey uses `github.com/cockroachdb/errors` with `errors.Wrapf` / `errors.Join`.

### 3.3 Naming

Keep names concise. Prefer meaningful short names. Avoid unnecessary abbreviations and verbose names.

Avoid:

```go
tmpData
managerObject
projectConfiguration
```

Prefer:

```go
cfg
proj
mgr
conn
```

### 3.4 Architecture

Prefer modifying existing code instead of introducing new abstractions.

- Do not introduce new structs or interfaces unless there is a clear benefit.
- Keep APIs stable.
- Minimize public surface.
- Favor composition over inheritance-like patterns.
- Do not repeat inherited conditions (e.g. `when`) at every level; declare them at the highest applicable scope.

### 3.5 Go Conventions

- Package aliases: `kkcorev1`, `kkcorev1alpha1`, `kkprojectv1`.
- `pkg/const` is imported as `_const` to avoid keyword collision.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubesphere/kubekey](https://github.com/kubesphere/kubekey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->

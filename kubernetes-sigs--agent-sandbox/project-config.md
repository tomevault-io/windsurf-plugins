---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Human contributors should also read [CONTRIBUTING.md](CONTRIBUTING.md), [docs/development.md](docs/development.md), and [docs/testing.md](docs/testing.md), which are the source of truth.
---

# AGENTS.md

Guidance for AI coding agents working in this repository. Human contributors should also read [CONTRIBUTING.md](CONTRIBUTING.md), [docs/development.md](docs/development.md), and [docs/testing.md](docs/testing.md), which are the source of truth.

## Project summary

`agent-sandbox` is a Kubernetes controller that provides the `Sandbox` CRD: a stateful, singleton, pod-backed workload with a stable identity, intended for AI agent runtimes, dev environments, notebooks, and similar use cases. It is a SIG Apps subproject (`sigs.k8s.io/agent-sandbox`) and follows Kubernetes / `controller-runtime` conventions.

- API group `agents.x-k8s.io/v1beta1`: `Sandbox` (core).
- API group `extensions.agents.x-k8s.io/v1beta1`: `SandboxClaim`, `SandboxTemplate`, `SandboxWarmPool` (opt-in extensions).
- Go module: `sigs.k8s.io/agent-sandbox` (Go 1.26.x, see [go.mod](go.mod)).

## Repository layout

| Path | What lives there |
| --- | --- |
| [**.agents/skills/**](.agents/skills/) | Specialized instructions for you (AI agents). See "Agent Skills" section below. |
| [api/v1beta1/](api/v1beta1/) | Core `Sandbox` types and kubebuilder markers. |
| [extensions/api/v1beta1/](extensions/api/v1beta1/) | `SandboxClaim`, `SandboxTemplate`, `SandboxWarmPool` types. |
| [controllers/](controllers/) | Core `Sandbox` reconciler + tests. |
| [extensions/controllers/](extensions/controllers/) | Reconcilers for the extension CRDs. |
| [cmd/agent-sandbox-controller/](cmd/agent-sandbox-controller/) | Controller manager entrypoint. |
| [internal/](internal/) | Shared internals: `lifecycle`, `metrics`, `version`. Not importable by external consumers. |
| [k8s/](k8s/) | Generated CRDs ([k8s/crds/](k8s/crds/)), RBAC, controller manifests. |
| [clients/k8s/](clients/k8s/) | **Generated** Kubernetes-style clientset, listers, informers (output of `dev/tools/client-gen-go.sh`). Do not hand-edit. |
| [clients/go/](clients/go/) | Hand-written high-level Go SDK that wraps the `SandboxClaim` lifecycle and exposes Gateway / port-forward / direct connectivity. Editable. |
| [clients/python/agentic-sandbox-client/](clients/python/agentic-sandbox-client/) | Hand-written Python SDK. Directory is named `agentic-sandbox-client` but the package publishes to PyPI as **`k8s-agent-sandbox`** — that's the name to import in docs and examples. |
| [examples/](examples/), [extensions/examples/](extensions/examples/) | Runnable sample manifests and demo apps. |
| [test/e2e/](test/e2e/), [test/benchmarks/](test/benchmarks/) | End-to-end and benchmark suites (Go-driven; some scenarios shell out via the Python SDK). |
| [dev/tools/](dev/tools/) | Repo tooling (lint, generate, deploy-kind, release scripts). Most `make` targets shell out here. |
| [dev/ci/](dev/ci/) | Prow presubmit/periodic scripts. |
| [docs/](docs/) | Development, testing, configuration docs and KEPs ([docs/keps/](docs/keps/)). |
| [site/](site/) | Hugo + Docsy source for https://agent-sandbox.sigs.k8s.io. Many pages are thin wrappers that `include-file` from the repo via mounts in [site/hugo.yaml](site/hugo.yaml) — see "Docs site mounts" below. Native page sources (lifecycle, snapshots, use-cases, runtime-templates, getting started, etc.) live only here. |

When in doubt about ownership, check the nearest `OWNERS` file.

## Agent Skills

This repository provides specialized instructions for AI agents in the standard [`.agents/skills/`](.agents/skills/) directory, following the Agent Skills specification.

## Build, test, lint

All standard tasks go through the [Makefile](Makefile). Prefer `make` targets over invoking tools directly so CI and local runs stay consistent.

- `make all` — runs `fix-go-generate`, `fix-api-docs`, `build`, `lint-go`, `lint-api`, `test-unit`, `toc-verify`, `verify-olm`. Run this before sending a PR.
- `make build` — compiles `bin/manager` from `cmd/agent-sandbox-controller`.
- `make test-unit` — runs Go unit tests with `-race` enabled and the Python unit test suites via `dev/tools/test-unit`.
- `make test-e2e` / `make test-e2e-race` — e2e suite against a kind cluster (much slower; e2e is not raced by default).
- `make lint-go` / `make fix-go` — `golangci-lint` (config in [dev/tools/.golangci.yaml](dev/tools/.golangci.yaml)).
- `make lint-api` / `make fix-api` — KAL API linter for kubebuilder tags and CRD conventions.
- `make toc-verify` / `make toc-update` — keep markdown TOCs in sync.
- `make deploy-kind` — create a local kind cluster named `agent-sandbox`, build images, deploy the controller, and write the kubeconfig to `bin/KUBECONFIG` (which the e2e suite expects). `EXTENSIONS=true make deploy-kind` to include the extension controllers; `CONTROLLER_ARGS="..."` passes flags to the controller; `CONTROLLER_ONLY=true` builds/pushes only the controller image (skipping example sidecars). `make delete-kind` tears it down.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubernetes-sigs/agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

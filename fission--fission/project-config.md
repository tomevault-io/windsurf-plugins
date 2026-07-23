---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

## CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Fission is a Kubernetes-native serverless framework, written in Go (`github.com/fission/fission`, Go 1.26).
The control plane is shipped as a single multi-headed binary (`fission-bundle`) that runs one of several subsystems based on flags.
Functions execute inside per-environment pods that the executor manages; the user-facing CLI is `fission`.

## Common commands

Build / lint / test (run from repo root):
- `make build-fission-cli` — build the `fission` CLI via goreleaser (snapshot, single target).
  `make install-fission-cli` copies it to `/usr/local/bin/fission`.
- `make code-checks` — `golangci-lint run` (config in `.golangci.yaml`; goimports local prefix `github.com/fission/fission`), plus `make verify-gomod`.
- `make verify-gomod` — fails if `go.mod` does not keep direct and indirect requirements in separate blocks (`go mod tidy` does NOT enforce this); runs in CI's "Verify dependencies" step.
  See "Dependency management".
- `make license` — add the SPDX header (`SPDX-FileCopyrightText` + `SPDX-License-Identifier: Apache-2.0`) to any in-scope source file (`.go`/`.sh`/`.py`/`Dockerfile*`) missing one.
  `make license-check` is the CI gate (runs in `lint.yaml`); run it before pushing.
- `make test-run` — runs `hack/runtests.sh`: pulls `setup-envtest` Kubebuilder assets for k8s 1.32.x, then `go test -race -coverprofile=coverage.txt ./...`.
  Requires `KUBEBUILDER_ASSETS` (the script sets it).
- `make check` — full local gate: `test-run` + `build-fission-cli` + `clean`.
- Run a single Go test: `go test -race -run TestName ./pkg/router/...` (set `KUBEBUILDER_ASSETS=$(go tool setup-envtest -p path use 1.32.x)` first if the package needs envtest).

Code generation (run after editing `pkg/apis/core/v1/types.go`):
- `make codegen` — regenerates clientset/listers/informers under `pkg/generated/` (via `hack/update-codegen.sh`) and deepcopy via controller-gen.
  Never hand-edit `pkg/generated/` or `zz_generated_*.go`.
- `make generate-crds` — regenerates CRD YAMLs under `crds/v1/` from the Go types.
- `make generate-webhooks` — regenerates webhook configs into the Helm chart from `// +kubebuilder:webhook` markers in `pkg/webhook/`.
- `make all-generators` — runs all generators including swagger and CLI/CRD ref docs.

Local cluster development (skaffold + kind):
- `kind create cluster --config kind.yaml` then `kubectl create ns fission && make create-crds`.
- `SKAFFOLD_PROFILE=kind make skaffold-deploy` — builds linux/amd64 images via goreleaser, copies per-binary Dockerfiles into `dist/*_linux_amd64_v1/`, and Helm-installs `charts/fission-all`.
  Other profiles: `kind-debug` (pprof + debugEnv), `kind-ci` (full observability), `kind-opentelemetry`.

Integration tests (`test/integration/`, Go + testify, build tag `//go:build integration`, expect a running Fission cluster reachable via `KUBECONFIG`):
- No `kubectl port-forward` needed: the framework registers `router.fission` / `router-internal.fission` / `mcp.fission` in a go-portless registry that port-forwards in-process (SPDY, per-dial pod re-resolution) from the `FISSION_NAMESPACE` namespace (default `fission`).
  Setting `FISSION_ROUTER` / `FISSION_ROUTER_INTERNAL` / `FISSION_MCP_BASE_URL` overrides a route with a fixed address (for hand-managed forwards or non-default installs).
- `/fission-function/<ns>/<name>` moved off the public listener after GHSA-3g33-6vg6-27m8 (see Architecture).
  Tests that invoke functions go through the framework's `Router(t)` HTTP client which auto-routes those paths to the internal listener, or dial `f.RouterInternalBaseURL()` via `f.HTTPClient()` — base-URL hosts are portless route names, so plain `http.Client`s cannot resolve them.
- Export `FISSION_INTERNAL_AUTH_SECRET` (read from `kubectl get secret fission-internal-auth -n fission -o jsonpath='{.data.secret}' | base64 -d`) so the framework's transport signs requests on the internal listener — leave unset to test the verifier's pass-through mode.
- The MCP test (`TestMCPToolsListAndCall`) needs `svc/mcp` in the cluster (`mcp.enabled`/`mcp.allowInsecure` are on in the kind/kind-ci skaffold profiles); it `t.Skip`s when the endpoint is unreachable.
- Run the full suite: `go test -tags=integration -timeout=30m -parallel 6 -v ./test/integration/suites/common/...`.
  Set runtime/builder image env vars (`NODE_RUNTIME_IMAGE`, `PYTHON_RUNTIME_IMAGE`, etc.) — tests `t.Skip` when their required image is unset.
  `TEST_NOCLEANUP=1` leaves resources for debugging.
- Run a single test: `go test -tags=integration -run TestNodeHelloHTTP -v ./test/integration/suites/common/...`.
- `suites/serial/` holds tests that mutate cluster-wide control-plane state (e.g. restarting the executor to exercise `AdoptExistingResources`) and so cannot run alongside the parallel `common` suite.
  CI runs them after `common/` in the same step, single-package: `go test -tags=integration -p 1 ./test/integration/suites/serial/...`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fission/fission](https://github.com/fission/fission) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

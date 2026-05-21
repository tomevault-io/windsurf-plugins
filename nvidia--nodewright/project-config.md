---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, Cursor, Codex, etc.) when working with code in this repository.
---

# CLAUDE.md / AGENTS.md

This file provides guidance to coding agents (Claude Code, Cursor, Codex, etc.) when working with code in this repository.

The canonical file lives at `.claude/CLAUDE.md`. The root-level `AGENTS.md` is a symlink to this file so agents that expect the cross-agent `AGENTS.md` convention find the same content.

## Repository overview

Skyhook (being renamed to NodeWright) is a Kubernetes-aware package manager for safely modifying host infrastructure at scale. It coordinates the node lifecycle (cordon → drain → apply package → interrupt/reboot → uncordon) as controlled rollouts gated by interruption budgets and deployment policies.

Rename status: the project is transitioning from Skyhook → NodeWright. Most public names (CRDs `skyhook.nvidia.com/v1alpha1`, CLI `kubectl skyhook`, namespace `skyhook`) still use `skyhook`. Components already moved to `nodewright`: the Go module (`github.com/NVIDIA/nodewright/operator`), the Helm chart (`name: nodewright`, distributed at `oci://ghcr.io/nvidia/nodewright/charts/nodewright`), and the operator image (`ghcr.io/nvidia/nodewright/operator`). The agent image is still at `ghcr.io/nvidia/skyhook/agent` pending its migration. Don't "fix" `nodewright` references back to `skyhook`, and don't preemptively rename what hasn't moved yet.

## Required reading: `docs/` (load every session)

`docs/` is the authoritative source for domain concepts and behavioral contracts — most traps in this codebase are explained there, not in code comments. **At the start of every session, before writing any code in this repo, read all of the files below into context.** The whole set is ~2000 lines / ~100KB — negligible for modern context windows, and skipping it almost guarantees a wrong-shaped PR that violates an unstated contract.

Required (read on session start):

- `docs/README.md` — docs index
- `docs/operator-status-definitions.md` — **Status / State / Stage** vocabulary (distinct concepts; conflating them causes subtle bugs)
- `docs/interrupt_flow.md` — cordon / drain / interrupt / uncordon sequence and `podNonInterruptLabels` semantics
- `docs/runtime_required.md` — runtime-required taint flow and `AutoTaintNewNodes`
- `docs/ordering_of_skyhooks.md` — priority, sequencing (`node` vs `all`), `SKYHOOK_NODE_ORDER`
- `docs/deployment_policy.md` — cluster-scoped rollout shaping
- `docs/resource_management.md` — strict resource-override validation (all 4 fields or none)
- `docs/versioning.md` — per-component semver, strictly enforced
- `docs/taints.md` — taint semantics
- `docs/providing_secrets_to_packages.md` — how packages consume secrets
- `docs/kubernetes-support.md` — supported K8s versions
- `docs/operator_resources_at_scale.md` — scale characteristics
- `docs/release-process.md`, `docs/releases.md` — release process and history
- `docs/cli.md` — CLI reference (needed before touching `operator/cmd/cli/`)

Read on demand (not required up-front):

- `docs/designs/` — design docs for existing features (consult before changing an existing feature)
- `docs/plans/` — in-flight plans (check before starting new work in an area)
- `docs/kyverno/`, `docs/metrics/` — policy and observability surfaces

If a doc above is silent on a question you need to answer, say so explicitly rather than guess.

## Three components, three toolchains

- **`operator/`** — Go controller-manager (controller-runtime, Kubebuilder v4). Go 1.26.3, vendored (`GOFLAGS=-mod=vendor`). Also hosts the CLI (`cmd/cli`) built as a kubectl plugin.
- **`agent/skyhook-agent/`** — Python 3.10+ package (hatch-managed). Runs inside every package container; reads `/skyhook-package/config.json` and executes lifecycle steps (apply / config / interrupt / post-interrupt / upgrade / uninstall). Tests via pytest, vendored deps under `agent/vendor/`.
- **`chart/`** — Helm chart. Generated from `operator/config/` via `helmify` (`make generate-helm`) but hand-edited after; don't regenerate blindly.

The root `Makefile` just fans out into `operator/` and `agent/` subdirectories. Most real targets live in `operator/Makefile`.

## Common commands

All operator commands run from `operator/`:

```bash
make build              # build manager + CLI (also runs manifests/generate/fmt/vet/lint)
make build-manager      # operator binary only → bin/manager
make build-cli          # kubectl-skyhook → bin/skyhook

make unit-tests         # ginkgo unit tests + envtest (fake apiserver), writes to reporting/
make test               # full suite: manifests, generate, fmt, vet, lint, unit + e2e + cli-e2e + helm + operator-agent
make e2e-tests          # chainsaw e2e against current cluster (set POOL=<name> to run one pool — see docs/ci-test-pools.md)
make watch-tests        # ginkgo watch mode

make run                # runs controller as background process against current kubeconfig (ENABLE_WEBHOOKS=false by default)
make kill               # stops the backgrounded manager

make manifests          # regenerate CRDs/RBAC/webhooks from kubebuilder markers — REQUIRED after editing api/
make generate           # regenerate zz_generated.deepcopy.go — REQUIRED after editing api/ types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/nodewright](https://github.com/NVIDIA/nodewright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

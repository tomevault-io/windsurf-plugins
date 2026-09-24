---
trigger: always_on
description: This file provides guidance to Claude Code, Codex, Cursor and other coding agents working in this repository. It is the canonical agent instruction file — `.github/copilot-instructions.md` points here rather than duplicating it.
---

# AGENTS.md

This file provides guidance to Claude Code, Codex, Cursor and other coding agents working in this repository. It is the canonical agent instruction file — `.github/copilot-instructions.md` points here rather than duplicating it.

## Local Overlay

If present, also read `AGENTS.local.md` at the repo root. It is gitignored, so personal overlays stay local — check the exact path directly (`Read` or `cat`), not via ignore-respecting tools such as `rg`, `fd`, or `git ls-files`. Follow it where it does not conflict with this file.

## Role & Expertise

Act as a Principal Engineer working on production Kubernetes infrastructure in Go. NVSentinel takes GPU nodes out of service and reboots them in live clusters, so a wrong decision here evicts real customer workloads. Favour correctness and operational safety over cleverness. All code must be production-grade, not illustrative.

## Project Overview

NVSentinel is a GPU node resilience system for Kubernetes. It detects, classifies and remediates hardware and software faults on GPU nodes.

**Pipeline:** Detect → Ingest → Act

```text
 Health Monitors             Ingestion                 Fault Management
┌───────────────────┐                            ┌───────────────────────┐
│ GPU (DCGM)        │                            │ Fault Quarantine      │
│ Syslog            │  gRPC   ┌──────────────┐   │  (cordon / taint)     │
│ CSP               │────────▶│   Platform   │   │ Node Drainer          │
│ NIC               │         │  Connectors  │   │  (evict)              │
│ Kubernetes Object │         └───────┬──────┘   │ Fault Remediation     │
│ Health Events     │                 │ persist  │  (creates maint. CR)  │
│   Analyzer        │                 ▼          └───────────┬───────────┘
└───────────────────┘      ┌────────────────────┐      ▲     │ maintenance
                           │    Event Store     │──────┘     │     CR
                           │ MongoDB/PostgreSQL │  reconcile ▼
                           └────────────────────┘   ┌──────────────────┐
                                                    │ Janitor          │
                                                    │  (reset/reboot)  │
                                                    └──────────────────┘
```

**Every health monitor is peer to every other one.** GPU, syslog, CSP, NIC, Kubernetes Object Monitor and the Health Events Analyzer are all just health monitors — the analyzer detects patterns across events rather than reading a device, but it has no special status in the pipeline and publishes over the same gRPC interface as the rest. Treat "add a monitor" as the same shape of task regardless of which one you are looking at.

Platform-connectors persists events to the store (MongoDB with change streams, or PostgreSQL) and updates node conditions on the Kubernetes API. Fault Quarantine, Node Drainer and Fault Remediation reconcile from the store and act on the cluster. Janitor is driven differently — it reconciles the maintenance CRs that Fault Remediation creates, via the Kubernetes API rather than the store.

**No module calls another module directly.** Coordination happens through the shared event store and the Kubernetes API. A change that introduces a direct call between two modules is almost certainly wrong — check [docs/designs/](docs/designs/) before proposing one.

**Tech stack:** Go 1.27.0, Python 3.10+ (Poetry), Kubernetes, gRPC + protobuf, MongoDB / PostgreSQL, Helm, DCGM. Tool versions are pinned in `.versions.yaml` — that file is the single source of truth; read it with `make show-versions`, never hardcode a version elsewhere.

## Commands

```bash
# THE gate. Run this before every PR — it is what CI runs.
make lint-test-all   # protos-lint + license-headers-lint + gomod-lint + all Go/Python/Helm/shell lint+test

# Individual module (every Go module has the same interface via make/go.mk)
make -C labeler lint-test    # vet + lint + test for one module
make -C labeler vet          # go vet ./...
make -C labeler lint         # golangci-lint with the repo .golangci.yml
make -C labeler test         # gotestsum, race detector on
make -C labeler coverage     # coverage report

# Single test
cd labeler && go test -race -run TestKataLabelDetection ./...

# Local cluster (ctlptl-managed Kind + registry, driven by Tilt)
make dev-env         # create cluster + start Tilt
make dev-env-clean   # stop Tilt + delete cluster
make dev-restart     # restart Tilt without recreating the cluster
make e2e-test        # end-to-end suite against the local cluster

# Codegen and hygiene — run after touching protobufs or go.mod
make protos-generate      # regenerate Go + Python protobuf bindings
make dependencies-sync    # sync deps across modules via the Go workspace
make go-mod-tidy-all      # go mod tidy in every module
make license-headers-lint # Apache 2.0 headers on all source files

# Images
make ko-build     # Go images (ko — no Dockerfile involved)
make docker-all   # Dockerfile-based images (Python, shell, CUDA/DCGM-based)

make help         # every target
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/NVSentinel](https://github.com/NVIDIA/NVSentinel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

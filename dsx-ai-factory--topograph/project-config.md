---
trigger: always_on
description: This file provides guidance to Codex, Cursor, Copilot, and other coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex, Cursor, Copilot, and other coding agents when working with code in this repository.

<!-- AUTO-SYNCED: canonical source is .claude/CLAUDE.md. The first 5 lines differ, and the relative links here resolve from the repository root, where .claude/CLAUDE.md prefixes the same links with ../ because it sits one directory down. Do not "restore" that difference. -->

## Start here

This file is the short core. It carries the invariants an agent must not break, the
boundaries of what an agent may change on its own, and the conventions that are
specific to this codebase. Everything else lives in the document that already owns
it and is not repeated here:

| You need | Read |
|---|---|
| Prerequisites, build, local test loop, running a binary, container images, packaging, CI parity | [DEVELOPMENT.md](DEVELOPMENT.md) |
| Issue-first workflow, DCO sign-off, commit and branch conventions, review process, AI-assisted contribution policy | [CONTRIBUTING.md](CONTRIBUTING.md) |
| What each runtime component does and how a request flows between them | [docs/architecture.md](docs/architecture.md) |
| Every label and annotation key the Kubernetes engine writes | [docs/reference/node-labels.md](docs/reference/node-labels.md) |
| Provider selection, supported providers, the "Choosing a Provider" scenario table | `docs/overview.md` |
| API endpoints, request parameters, response fields, config schema | `docs/api.md` |
| How to report a suspected vulnerability | `SECURITY.md` |
| Release history and operator-facing migration notes | `CHANGELOG.md` |

When you change behaviour that one of those documents describes, update that
document. Do not answer the same question here as well; two copies of a rule drift
apart, and the next agent reads the stale one.

## 1. Project Overview and Architecture

Topograph discovers the physical network topology of a cluster (NVLink domains,
InfiniBand/Ethernet switch fabric, cloud rack topology) and exposes it to workload
schedulers: Slurm, Kubernetes, and Slurm-on-Kubernetes (Slinky). It has five runtime
components, the API Server, the Node Observer, the Node Data Broker, the Provider,
and the Engine. [docs/architecture.md](docs/architecture.md) describes what each one
does and how a request flows between them.

### Key invariant

Providers differ by environment. The canonical `topology.Graph` is stable. Engines
only translate; they do not discover.

Within a provider, network-fabric and accelerator-domain discovery may be composed
independently through `pkg/accelerator`; the provider remains responsible for
combining both dimensions into the canonical graph.

This separation is load-bearing. If you find yourself reading the fabric in an
engine, or emitting scheduler-specific output from a provider, stop and reconsider.

### Repository map

```
cmd/                  # Entry points: topograph, node-observer, node-data-broker, kwok-nodes
pkg/
  accelerator/        # Pluggable accelerator-domain discovery composed by providers
  providers/          # One directory per provider: aws, crusoe, dra, dsx, gcp, infiniband, lambdai, nebius, netq, nscale, oci, test
  engines/            # One directory per engine: graph, k8s, nfd, slinky, slurm
  topology/           # Canonical Graph, Vertex tree, and topology constants (DO NOT CHANGE CASUALLY)
  registry/           # Central NamedLoader wiring for providers + engines
  translate/          # topology.conf and block/tree generation shared by engines
  server/             # HTTP server and request aggregator
  node_observer/      # Kubernetes Node watcher
  ib/                 # InfiniBand fabric discovery helpers
  config/             # Config file parser
  metrics/            # Prometheus metrics
  models/             # Go types and loader for YAML simulation models (the YAML files live in tests/models/)
  test/               # Cross-package test helpers
internal/             # Shared utilities not part of the public API
  cluset, component, config, exec, files, httperr, httpreq, k8s, kwok, version
charts/topograph/     # Helm chart for all Kubernetes components; tests/ holds the helm-unittest suites + snapshots
CHANGELOG.md          # Release history (Keep a Changelog format); update [Unreleased] for user-facing PRs
docs/                 # Public-facing docs: overview.md, architecture.md, api.md + providers/, engines/, reference/ subdirectories
demos/                # Interactive Kubernetes/KWOK deployment demos
tests/models/         # YAML simulation fixtures
config/               # Sample topograph-config.yaml
scripts/              # Build scripts (deb, rpm, SSL, clean)
localdev/             # Developer-local workspace, not tracked; personal scratch files
```

## 2. What an Agent Is Permitted to Do

### In scope without asking

An agent is permitted to make these changes on its own initiative, as long as the
change ships with tests when it changes behavior, keeps `make qualify` green, and
carries the doc updates named in the Documentation Impact Evaluation table below.

| Task in scope | Files an agent may modify |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dsx-ai-factory/topograph](https://github.com/dsx-ai-factory/topograph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->

---
trigger: always_on
description: Guidance for AI coding agents working in `mariadb-operator`. Read this before making changes.
---

# AGENTS.md

Guidance for AI coding agents working in `mariadb-operator`. Read this before making changes.

## Project Overview

`mariadb-operator` is a Kubernetes operator that manages the full lifecycle of MariaDB: provisioning, high availability (MariaDB native replication and Galera), multi-cluster topologies, backups (logical and physical), point-in-time recovery, users/grants/databases as code, MaxScale proxying, TLS, metrics and updates.

- **Module**: `github.com/mariadb-operator/mariadb-operator/v26`.
- **API group**: `k8s.mariadb.com/v1alpha1`
- **Stack**: Go, controller-runtime, Kubebuilder markers + controller-gen, Ginkgo v2/Gomega, envtest, KIND, Helm
- **Docs**: `docs/*.md` (feature-oriented, one file per topic — see Feature Map)

## Simplicity

This project is **highly biased towards simplicity and against over-engineering** — every line merged is a line maintained:

- Prefer the simplest implementation that solves the actual problem; reuse existing patterns before introducing new abstractions, layers or dependencies.
- Complexity without a clear outcome is a red flag — speculative flexibility and unrealistic corner cases included.
- New config options, spec fields, flags and dependencies multiply the test matrix and maintenance burden — justify them with a concrete use case.

In scope decisions and reviews, treat unjustified complexity as a defect.

### CRDs

| Kind | Purpose |
|------|---------|
| `MariaDB` | Main resource: MariaDB servers with optional replication/Galera HA, multi-cluster, maintenance, TLS, metrics, storage |
| `MaxScale` | MaxScale database proxy connected to a MariaDB |
| `Backup` | Logical backups (mariadb-dump) with S3/Azure/PVC storage, scheduling and retention |
| `PhysicalBackup` | Physical backups via mariadb-backup CLI or Kubernetes `VolumeSnapshot` |
| `Restore` | Restores a MariaDB from a backup (`bootstrapFrom` also embeds this logic) |
| `PointInTimeRecovery` | PITR: continuous binlog archival (data-plane agent) + recovery to a target time. Short name: `pitr` |
| `Database`, `User`, `Grant` | SQL resources reconciled against a MariaDB/ExternalMariaDB |
| `SqlJob` | Runs arbitrary SQL scripts as Jobs/CronJobs |
| `Connection` | Generates connection string Secrets, with health checks |
| `ExternalMariaDB` | Represents a MariaDB outside the cluster; enables SQL resources against it and multi-cluster membership |

## Repository Structure

```
mariadb-operator/
├── .agents/skills/          # Agent skills (SKILL.md per dir); .claude/skills is a symlink to it so both
│                            #   OpenCode and Claude Code auto-discover them
├── api/v1alpha1/            # CRD types, kubebuilder markers, defaults, indexes (*_indexes.go), helpers
├── cmd/                     # Entrypoints (single binary, cobra subcommands)
│   ├── controller/          # Root cmd = operator; subcommands: webhook, cert-controller
│   ├── agent/               # Data-plane sidecar agent (replication, galera subcommands)
│   ├── init/                # Init container (replication, galera subcommands)
│   ├── backup/              # Backup CLI (nested `restore` subcommand)
│   └── pitr/                # PITR CLI (binlog archive/replay)
├── internal/
│   ├── controller/          # Reconcilers + integration tests (envtest, suite_test.go)
│   ├── webhook/v1alpha1/    # Validation webhooks (one per CRD)
│   └── helmtest/            # Helm chart unit tests (terratest)
├── pkg/                     # Reusable libraries
│   ├── builder/             # Builds all child K8s objects (StatefulSets, Jobs, Services...)
│   ├── controller/          # Sub-reconcilers: auth, batch, certificate, configmap, deployment,
│   │                        #   endpoints, galera, maintenance, pvc, rbac, replication, secret,
│   │                        #   service, servicemonitor, sql, statefulset
│   ├── sql/                 # MariaDB SQL client (users, grants, replication, Galera, binlogs, locks)
│   ├── condition/           # Status condition helpers (Ready / Complete)
│   ├── refresolver/         # Resolves refs to Secrets, ConfigMaps, MariaDB, MaxScale...
│   ├── watch/               # WatcherIndexer for indexed watches
│   ├── discovery/           # Runtime API discovery (cert-manager, ServiceMonitor, VolumeSnapshot, SCC)
│   ├── environment/         # Operator/pod env config (RELATED_IMAGE_*, MARIADB_OPERATOR_*)
│   ├── replication/, galera/# HA orchestration logic
│   ├── webhook/             # Immutability validation via struct tags
│   ├── embed/               # Embedded mariadb-docker entrypoint (generated — do not edit)
│   └── ...                  # backup, binlog, pki, health, metadata, pod, wait, etc.
├── config/                  # Kustomize sources (crd/, rbac/, webhook/, manager/, default/, samples/)
├── deploy/                  # Generated distributables
│   ├── charts/              # Helm charts: mariadb-operator, mariadb-operator-crds, mariadb-cluster
│   ├── crds/                # Rendered crds.yaml
│   └── manifests/           # Rendered install bundles
├── docs/                    # Feature docs + generated api_reference.md
├── examples/                # Example manifests (incl. multi-cluster/) and Flux GitOps setup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mariadb-operator/mariadb-operator](https://github.com/mariadb-operator/mariadb-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

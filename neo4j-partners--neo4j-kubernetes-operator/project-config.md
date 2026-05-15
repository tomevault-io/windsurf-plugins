---
trigger: always_on
description: - Neo4j Enterprise Operator (Kubebuilder/controller-runtime 0.21, Go 1.24) automates Neo4j Enterprise 5.26.x (last semver LTS) and 2025.x.x+ (CalVer) on Kubernetes.
---

# AGENTS.md

## Quick Mission
- Neo4j Enterprise Operator (Kubebuilder/controller-runtime 0.21, Go 1.24) automates Neo4j Enterprise 5.26.x (last semver LTS) and 2025.x.x+ (CalVer) on Kubernetes.
- Status: **alpha**; expect churn. Follow `docs/` plus historical reports in `reports/`.
- Platform assumptions: Kubernetes ≥1.21, cert-manager ≥1.18 for TLS, and **Kind-only** for any cluster work.

## Non-Negotiables (read with `CLAUDE.md`)
1. **Kind only**: use `make dev-cluster`, `test-cluster`, `operator-setup`; never minikube/k3s.
2. **Enterprise images only** (`neo4j:<version>-enterprise`). Discovery uses LIST resolver with static pod FQDNs (port 6000); 5.26.x requires explicit `V2_ONLY`, CalVer 2025.x+ (including 2026.x+) omits the flag — handled in `buildVersionSpecificDiscoveryConfig()` via `isCalverImage()` / `ParseVersion`.
3. **Operator must run in-cluster**: never `make dev-run`/`hack/dev-run.sh` or host-mode runs.
4. **Server-based design**: single `{cluster}-server` StatefulSet with `topology.servers`; preserve `topology.serverModeConstraint/serverRoles` hints and centralized `{cluster}-backup` StatefulSet.
5. **Conflict-safe writes**: wrap creates/updates in `retry.RetryOnConflict`; when checking existing resources use UID (not ResourceVersion) for template comparison.
6. **Edition removed**: API assumes Enterprise; do not reintroduce the field or allow community images.
7. **Safety hooks**: keep split-brain detector (`internal/controller/splitbrain_detector.go`) wiring and event reason `SplitBrainDetected`; status `.phase` drives readiness checks.
8. **Plugin rules**: APOC via env vars; Bloom/GDS/GenAI and similar via ConfigMap with automatic security defaults and dependency resolution; respect cluster vs standalone naming/labeling in `plugin_controller`.
9. **Property sharding**: `Neo4jShardedDatabase` + related tests stay opt-in; requires ≥5 servers with 4–8Gi each—guard resource gates.
10. **Database & TLS**: `Neo4jDatabase` must work for cluster and standalone (ensure `NEO4J_AUTH` for standalone), respect seedURI/seedConfig, TLS automation for `spec.tls.mode=cert-manager`. Discovery settings differ by version — always go through `buildVersionSpecificDiscoveryConfig()`, never hard-code K8S or V1 discovery settings.
11. **CRD scope separation**: Cluster/Standalone manage infra/config; Database manages database lifecycle only—no cross-CRD overrides.

## Architecture Anchors
- **CRDs (`api/v1beta1`)**: Neo4jEnterpriseCluster, Neo4jEnterpriseStandalone, Neo4jDatabase, Neo4jPlugin, Neo4jBackup, Neo4jRestore, Neo4jShardedDatabase.
- **Controllers (`internal/controller/`)**:
  - Cluster/Standalone reconcilers build ConfigMaps/Services/StatefulSets, manage status phases, TLS, auth, placement, and cache/configmap managers.
  - Database controller auto-detects cluster vs standalone, uses appropriate Bolt client, and supports wait/ifNotExists/topology/seed flows.
  - Plugin controller manages env-vs-config installs, dependencies, and pod readiness per deployment type.
  - Backup controller runs centralized `{cluster}-backup` pods with request file drop; Restore controller handles PIT restores.
  - Sharded database controller enforces property-sharding readiness; Topology scheduler adds AZ spread/anti-affinity; Rolling upgrade orchestrator handles leader-aware upgrades and metrics.
  - Split-brain detector restarts orphans after multi-pod view comparison.
- **Validation (`internal/validation/`)**: auth, backup, cluster, database (cluster+standalone), image, memory, plugin, resource, security, shardeddatabase, storage, TLS, topology, upgrade validators; edition validator stubbed out.
- **Resources/Clients**: `internal/resources` builds Kubernetes objects (TLS policies, discovery ports, memory sizing); `internal/neo4j` wraps Bolt, version parsing (5.x vs 2025.x), and upgrade safety checks.

## Repository Atlas
| Area | Purpose |
| --- | --- |
| `cmd/main.go` | Manager entrypoint wiring controllers/webhooks. |
| `api/v1beta1/` | CRD schemas listed above. |
| `internal/controller/` | Reconcilers, split-brain detector, topology scheduler, rolling upgrade, cache/configmap managers. |
| `internal/resources/` | Builders for StatefulSets/Services/ConfigMaps, memory sizing, TLS/discovery helpers. |
| `internal/neo4j/` | Bolt client + version helpers used by controllers/tests. |
| `internal/validation/` | Validation/recommendation logic per CRD. |
| `charts/neo4j-operator/` | Helm chart (README quick start). |
| `config/` | Kubebuilder manifests (CRDs, RBAC, samples, overlays). |
| `docs/` | User/developer guides, deployment/seed-URI/split-brain guides, quick reference, API reference. |
| `examples/` | Standalone, clusters, backup/restore, plugins, property sharding, E2E scenarios. |
| `test/` | Ginkgo integration suites, fixtures, helpers. |
| `scripts/` | Automation (`test-env.sh`, setup/cleanup, demos, RBAC helpers, verification). |
| `reports/` | Design history and implementation analyses. |

## Development Workflow
1. Prereqs: Go 1.24+, Docker, kubectl, Kind, make, git; verify `kind version`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neo4j-partners/neo4j-kubernetes-operator](https://github.com/neo4j-partners/neo4j-kubernetes-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->

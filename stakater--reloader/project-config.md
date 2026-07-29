---
trigger: always_on
description: Reloader is a Kubernetes operator that automatically triggers rolling restarts of workloads when the ConfigMaps or Secrets they reference are updated. Without it, Kubernetes does not restart pods when configuration changes — operators must do it manually or rely on GitOps pipelines.
---

# Stakater Reloader Project Memory

## Project Purpose

Reloader is a Kubernetes operator that automatically triggers rolling restarts of workloads when the ConfigMaps or Secrets they reference are updated. Without it, Kubernetes does not restart pods when configuration changes — operators must do it manually or rely on GitOps pipelines.

**What it watches**: ConfigMaps, Secrets, Namespaces, and (optionally) `SecretProviderClassPodStatus` (CSI-mounted secrets).

**Workload types it can reload**: Deployment, StatefulSet, DaemonSet, CronJob, Job, Argo Rollout, and OpenShift DeploymentConfig.

**How restarts are triggered**: Two strategies (selected via `--reload-strategy`):
1. **env-vars** (default) — injects an environment variable (`STAKATER_{NAME}_{TYPE}`) into every container with the SHA1 hash of the resource's data. A change in data changes the env var value, causing Kubernetes to restart pods.
2. **annotations** — writes the SHA1 hash into the pod template's annotations, which also forces a rollout.

**The core problem it solves**: ConfigMaps and Secrets are decoupled from pod lifecycle in Kubernetes. Applications reading config at startup see stale data after a config update unless pods are restarted. Reloader closes that gap automatically and selectively.

**Potential improvements observed**:
- **Duplicate reload suppression**: If a workload references both a ConfigMap and a Secret that are updated in the same controller reconcile cycle, it may get reloaded twice. Could be solved with a per-workload debounce map keyed by namespace/name/resourceVersion, flushed after a short TTL.
- **CronJob/Job reload is destructive**: Jobs are deleted and recreated on change, which loses run history. Could instead only annotate the CronJob template without spawning a new Job.
- **No per-resource reload rate limiting**: A rapid-fire ConfigMap update (e.g., from a CI pipeline) can trigger many restarts. A cooldown window per resource would help.
- **CSI integration gap**: CSI volumes are watched at the `SecretProviderClassPodStatus` level, but the link back to the workload is indirect and may miss edge cases. Needs a direct map from SecretProviderClass → workloads that mount it.

---

## Repo Map

| Path | Owns | Inspect when |
|---|---|---|
| `main.go` | Entry point, delegates to `app.Run()` | Never needs changes |
| `internal/pkg/app/` | `Run()` bootstrap, Cobra command wiring | Startup sequence changes |
| `internal/pkg/cmd/` | CLI flags parsing, `startReloader()`, controller/HA wiring | Adding new flags or startup behavior |
| `internal/pkg/controller/` | Informer/queue per resource type, event handlers (Add/Update/Delete) | Watching new resource types, queue tuning |
| `internal/pkg/handler/` | Per-event handlers (create, update, delete), `doRollingUpgrade()`, pause deployment | Core reload logic changes |
| `internal/pkg/callbacks/` | Workload-specific get/list/update/patch functions, `RollingUpgradeFuncs` struct | Adding new workload types |
| `internal/pkg/options/` | All CLI flag variables, defaults, `ArgoRolloutStrategy` type | Adding or renaming flags |
| `internal/pkg/constants/` | Constants: env var postfixes, annotation prefix, strategy names, HA lock name | Renaming global identifiers |
| `internal/pkg/metrics/` | Prometheus `Collectors` struct, all metric registration and recording helpers | Adding metrics |
| `internal/pkg/alerts/` | Slack/Teams/GChat/raw webhook alerting, env var config | Alert sink changes |
| `internal/pkg/util/` | SHA generation via `crypto/sha.go`, env var name conversion, namespace/label utilities | Utility/hash changes |
| `internal/pkg/crypto/` | `GenerateSHA(data)` — SHA1 hex digest | Hash algorithm changes |
| `internal/pkg/leadership/` | Leader election via Kubernetes Lease, HA stop/start of controllers | HA behavior changes |
| `internal/pkg/testutil/` | Fake Kubernetes objects for unit tests | Writing new tests |
| `pkg/common/` | `ReloadCheckResult`, `ReloaderOptions`, `ShouldReload()` logic, `Config` struct | Reload decision logic, annotation precedence |
| `pkg/kube/` | `Clients` struct (k8s + OpenShift + Argo + CSI), `GetKubernetesClient()`, `ResourceMap` | Client initialization, new CRD clients |
| `deployments/` | Helm chart (`deployments/kubernetes/chart/reloader/`), Kustomize manifests | Helm values, RBAC, deployment config |
| `docs/` | User-facing annotation documentation, architecture notes | Writing docs or confirming annotation behavior |
| `scripts/` | Shell scripts used by CI and Makefile | Build/release pipeline |
| `test/loadtest/` | Load test CLI (`cmd/loadtest`), 13 scenarios (S1–S13), Kind cluster setup | Performance testing, regression benchmarks |
| `.github/` | CI workflows: lint, test, Kind e2e, multi-arch Docker build, release | CI changes |

---

## Core Runtime Flow

**1. Entry** — `main.go:10` calls `app.Run()`.

**2. CLI Init** — `internal/pkg/app/app.go` calls `cmd.NewReloaderCommand()` which registers all Cobra flags from `options/flags.go` and runs `startReloader()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stakater/Reloader](https://github.com/stakater/Reloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

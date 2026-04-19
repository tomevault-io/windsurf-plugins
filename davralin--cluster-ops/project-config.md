---
trigger: always_on
description: enableServiceLinks: false
---

# CLAUDE.md — cluster-ops conventions

GitOps Kubernetes cluster: Talos Linux, Flux, Cilium, Rook-Ceph, SOPS/Age.

## Commit & PR Style

- **Atomic commits** — each commit is one logical, self-contained change
- **Atomic PRs** — each pull request contains only related, grouped commits
- Use [Conventional Commits](https://www.conventionalcommits.org/): `type(scope): description`
  - `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`
- Keep commit messages concise; use the body for detail when needed
- Squash/rebase before merging if intermediate commits aren't meaningful
- **PR descriptions should be brief** — let the code speak for itself. Don't write a blog post for every pull request

## Repository Structure

```
kubernetes/
├── apps/          # Application deployments (one dir per app or namespace group)
├── clusters/      # Cluster-level Flux Kustomizations (postBuild variable substitution)
├── core/          # Core infra (Cilium, cert-manager, HAProxy, Flux)
├── monitoring/    # Prometheus stack, alerting rules
├── security/      # Admission policies (Kyverno)
└── storage/       # Rook-Ceph cluster configuration
ansible/           # Host provisioning playbooks (AWX)
talos/             # Talos Linux machine configs
```

## App Deployment Patterns

### Prefer app-template built-ins

Before creating separate resource files, check if app-template handles it natively:
- **RBAC** → `rbac.roles` + `rbac.bindings`
- **ConfigMaps** → `configMaps` (inline data, named `<fullnameOverride>-<key>`)
- **Raw resources** → `rawResources` (CiliumNetworkPolicies, etc.)
- **ServiceAccounts** → `serviceAccount: { <name>: {} }`

**Chart selection hierarchy:**
1. Dedicated upstream Helm charts — when available and they expose the values we need (securityContext, podSecurityContext are common gaps)
2. bjw-s app-template — for most apps
3. dysnix raw chart — for operator CRDs (VolSync ReplicationSources, CNPG Clusters, Prometheus Probes, etc.) that need their own HelmRelease lifecycle

### Standalone app (own namespace)

Directory: `kubernetes/apps/<appname>/`

Required files:
- `helm-release.yaml` — HelmRelease using bjw-s app-template
- `kustomization.yaml` — lists all resource files
- `namespace.yaml` — Namespace manifest
- `networkpolicy.yaml` — default-deny NetworkPolicy
- `probes.yaml` — blackbox exporter Probe (wrapped in dysnix raw chart)
- `pvc.yaml` — PersistentVolumeClaim (if app has persistent data)
- `volsync.yaml` — VolSync Restic backup (if app has persistent data)

If app needs a database, add:
- `helm-release-db-v18.yaml` — CNPG PostgreSQL cluster (PG version in filename)
- `database-secret.yaml` — SOPS-encrypted secret with DB credentials + S3 backup keys

If app needs secrets, add:
- `secret.yaml` — SOPS-encrypted Secret manifest

### Grouped apps (shared namespace)

Example: `kubernetes/apps/media/<appname>/`

- Apps share the parent namespace (e.g. `media`, defined in `kubernetes/apps/media/namespace.yaml`)
- Shared default-deny at parent level (`kubernetes/apps/media/networkpolicy.yaml`)
- Each sub-app has its own directory with `helm-release.yaml` and `kustomization.yaml`
- Parent `kustomization.yaml` lists all sub-app directories + shared resources

### Secrets pattern

Secrets use SOPS with Age encryption. When adding a new app that needs secrets:

1. Create `secret.yaml` (or `database-secret.yaml`) with **dummy placeholder values** — NOT real credentials
2. Do NOT add it to `kustomization.yaml` yet
3. The file serves as a template — fill in real values, encrypt with `sops -e -i secret.yaml`, then add to `kustomization.yaml`

This prevents accidental deployment of unencrypted secrets. CNPG won't bootstrap and Flux won't apply resources not listed in kustomization.yaml.

## HelmRelease Template

Every app uses bjw-s app-template. Copy from an existing app (e.g. `mealie`) and modify.

```yaml
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: &name appname
  namespace: flux-system          # ALWAYS flux-system
spec:
  interval: 30m
  chart:
    spec:
      chart: app-template
      version: 4.6.2             # Renovate manages this
      sourceRef:
        kind: HelmRepository
        name: bjw-s-charts
        namespace: flux-system
      interval: 30m
  driftDetection:
    mode: enabled
  targetNamespace: appname        # Actual namespace (or "media" for media apps)
  install:
    createNamespace: true
    remediation:
      retries: 10
  upgrade:
    remediation:
      retries: 10
  values:
    fullnameOverride: *name
    defaultPodOptions:
      automountServiceAccountToken: false
      enableServiceLinks: false
      securityContext:
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
        fsGroupChangePolicy: "OnRootMismatch"
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
    controllers:
      appname:
        containers:
          appname:
            image:
              repository: ghcr.io/example/app
              tag: v1.0.0@sha256:abc123   # Always pin digest — Renovate updates these
            env:
              TZ: "${TIMEZONE}"
            probes:
              liveness: &probes
                enabled: true
              readiness: *probes
              startup: *probes
            securityContext:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davralin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

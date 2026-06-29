---
trigger: always_on
description: This is a Kubernetes homelab GitOps monorepo managed with Flux v2 on Talos Linux.
---

# AGENTS.md — AI Agent Guide for home-ops

This is a Kubernetes homelab GitOps monorepo managed with Flux v2 on Talos Linux.
Read this file before making any changes.

---

## Project Overview

| Layer | Technology |
|---|---|
| OS | Talos Linux v1.12.4 (immutable, API-driven) |
| Kubernetes | v1.35.1 |
| GitOps | Flux v2 (flux-operator + flux-instance) |
| CNI | Cilium (BGP, native routing, kube-proxy replacement) |
| Ingress | Envoy Gateway (Kubernetes Gateway API) |
| Storage | Rook-Ceph (block) + OpenEBS (local hostpath) |
| Backup | VolSync + Kopia → NFS (singularity.milkyway) |
| Database | CloudNative-PG (PostgreSQL 18, HA) |
| Secrets | SOPS + Age + PGP |
| Helm charts | bjw-s/app-template (OCI) for nearly all apps |
| Updates | Renovate (hourly GitHub Actions) |
| Auth | Authelia + LLDAP |

---

## Repository Layout

```
kubernetes/
├── flux/cluster/ks.yaml        # Flux entrypoint → kubernetes/apps/
├── components/                 # Reusable Kustomize components
│   ├── common/                 # Namespace, OCI repos, SOPS secret, Flux alerts
│   ├── ext-auth/               # Authelia external auth (Envoy SecurityPolicy)
│   ├── nfs-scaler/             # KEDA autoscaler for NFS-dependent pods
│   ├── persistence/            # PVC + VolSync backup/restore templates
│   └── replacements/           # Shared variable substitution
└── apps/<namespace>/<app>/
    ├── ks.yaml                 # Flux Kustomization
    └── app/
        ├── kustomization.yaml
        ├── helmrelease.yaml
        └── secret.sops.yaml    # (optional) SOPS-encrypted secret
```

### Namespaces

`cert-manager`, `database`, `default`, `downloads`, `flux-system`, `home-automation`,
`kube-system`, `media`, `network`, `observability`, `openebs-system`, `rook-ceph`,
`security`, `system-upgrade`, `volsync-system`

---

## Universal App Pattern

Every app follows this exact two-file pattern. Study an existing app (e.g.,
`kubernetes/apps/default/vaultwarden/`) before adding a new one.

### `ks.yaml` — Flux Kustomization

```yaml
# yaml-language-server: $schema=https://k8s-schemas.home-operations.com/kustomize.toolkit.fluxcd.io/kustomization_v1.json
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: <app>
  namespace: flux-system
spec:
  targetNamespace: <namespace>
  commonMetadata:
    labels:
      app.kubernetes.io/name: <app>
  interval: 10m
  path: ./kubernetes/apps/<namespace>/<app>/app
  prune: true
  sourceRef:
    kind: GitRepository
    name: home-ops
  wait: true
  # Include ONLY the components that this app actually needs:
  components:
    - ../../../../components/persistence   # if app needs PVC + VolSync backup
    - ../../../../components/ext-auth      # if app needs Authelia auth
    - ../../../../components/nfs-scaler   # if app needs NFS (media/downloads)
  dependsOn:
    - name: rook-ceph-cluster              # if using ceph-block storage
      namespace: rook-ceph
    - name: cloudnative-pg                 # if using PostgreSQL
      namespace: database
    - name: volsync                        # if using persistence component
      namespace: volsync-system
  postBuild:
    substituteFrom:
      - kind: Secret
        name: cluster-secrets
    substitute:
      APP: <app>
      CAPACITY: 5Gi                        # if using persistence component
```

### `app/helmrelease.yaml` — HelmRelease (app-template)

```yaml
# yaml-language-server: $schema=https://k8s-schemas.home-operations.com/helm.toolkit.fluxcd.io/helmrelease_v2.json
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: <app>
spec:
  interval: 30m
  chartRef:
    kind: OCIRepository
    name: app-template
    namespace: flux-system
  install:
    remediation:
      retries: 3
  upgrade:
    cleanupOnFail: true
    remediation:
      strategy: rollback
      retries: 3
  values:
    controllers:
      <app>:
        annotations:
          reloader.stakater.com/auto: "true"
        containers:
          app:
            image:
              repository: <registry>/<image>
              tag: <version>@sha256:<digest>   # ALWAYS pin both tag AND digest
            env:
              TZ: Europe/Paris
            securityContext:
              allowPrivilegeEscalation: false
              readOnlyRootFilesystem: true
              capabilities: { drop: ["ALL"] }
            resources:
              requests:
                cpu: 10m
              limits:
                memory: 512Mi
        pod:
          securityContext:
            runAsNonRoot: true
            runAsUser: 568
            runAsGroup: 568
            fsGroup: 568
            fsGroupChangePolicy: OnRootMismatch
    service:
      app:
        controller: <app>
        ports:
          http:
            port: <port>
    ingress: {}        # NOT used — use HTTPRoute instead (see Networking below)
```

### `app/kustomization.yaml`

```yaml
# yaml-language-server: $schema=https://json.schemastore.org/kustomization
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - helmrelease.yaml
  - secret.sops.yaml   # only if secrets exist
```

---

## Networking

**Do NOT use `Ingress` resources.** This cluster uses Kubernetes Gateway API exclusively.

### Gateways

| Gateway | IP | Use for |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Diaoul/home-ops](https://github.com/Diaoul/home-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

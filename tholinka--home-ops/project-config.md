---
trigger: always_on
description: >
---


# home-ops Copilot Instructions

GitOps-managed Kubernetes homelab running Talos Linux. All infrastructure is declarative and managed through git. Flux CD reconciles the desired state from this repository.

---

## Project Layout

```
kubernetes/
├── flux/cluster/ks.yaml             # Flux entry point — points at kubernetes/apps/
├── apps/{category}/                  # One directory per namespace/category
│   ├── kustomization.yaml           # K8s Kustomization: sets namespace, lists app ks.yaml refs, includes components
│   └── {app}/
│       ├── ks.yaml                  # Flux Kustomization CRD(s) — entry point for this app
│       └── app/
│           ├── kustomization.yaml   # K8s Kustomization: lists the actual resources
│           ├── helmrelease.yaml     # Flux HelmRelease CRD
│           └── ...                  # PVCs, ExternalSecrets, ConfigMaps, etc.
├── components/                      # Reusable Kustomize Components (kind: Component)
│   ├── common/                      # Namespace creation, alerts, ExternalSecret for substitutions
│   ├── repos/app-template/          # OCIRepository for bjw-s app-template Helm chart
│   ├── volsync/                     # VolSync backup/restore (PVC + ReplicationSource/Destination)
│   ├── ext-auth/                    # Authentik external auth SecurityPolicy
│   ├── cnpg/{app,app-template}      # CloudNativePG database provisioning
│   ├── dragonfly/                   # Dragonfly (Redis alternative) — see its README for type selection
│   └── scaler/                      # HPA scale to zero variants (scales to zero if required service is missing) (instance, metrics, statefulset)
talos/
├── talconfig.yaml                   # Single source of truth — talhelper generates clusterconfig/ from this
├── talsecret.yaml                   # Encrypted secrets
├── patches/{global,controller}/     # Composable machine patches
└── clusterconfig/                   # GENERATED — never edit directly
bootstrap/
├── bootstrap-cluster.sh             # Main bootstrap entry point
├── lib/common.sh                    # Shared logging/talosctl helpers
└── helmfile.d/                      # Helmfile charts for initial bootstrap ONLY
```

---

## How Flux Discovers Apps

The reconciliation chain works as follows:

1. **`kubernetes/flux/cluster/ks.yaml`** — A Flux Kustomization pointing `path: ./kubernetes/apps` with patches that auto-apply HelmRelease defaults (drift detection, rollback, upgrade remediation) to all child Kustomizations.
2. **`kubernetes/apps/{category}/kustomization.yaml`** — A standard Kubernetes Kustomization (`kustomize.config.k8s.io/v1beta1`) that:
   - Sets `namespace:` for the category
   - Lists each app's `ks.yaml` as a resource
   - Includes shared components (typically `../../components/common` and `../../components/repos/app-template`)
3. **`kubernetes/apps/{category}/{app}/ks.yaml`** — One or more Flux Kustomization CRDs (`kustomize.toolkit.fluxcd.io/v1`). Simple apps have one document; complex apps (e.g., cnpg, envoy-gateway) use multiple `---`-separated documents for CRDs, operator, config, etc.
4. **`kubernetes/apps/{category}/{app}/app/kustomization.yaml`** — Standard Kubernetes Kustomization listing the actual resources (HelmRelease, PVCs, ExternalSecrets, etc.)

---

## Adding a New App

### 1. Create the app directory

```
kubernetes/apps/{category}/{app-name}/
├── ks.yaml
└── app/
    ├── kustomization.yaml
    └── helmrelease.yaml
```

### 2. Write the Flux Kustomization (`ks.yaml`)

```yaml
---
# yaml-language-server: $schema=https://schemas.tholinka.dev/kustomize.toolkit.fluxcd.io/kustomization_v1.json
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: &app my-app
  namespace: &namespace self-hosted
spec:
  interval: 1h
  components:
    - ../../../../components/volsync # if app needs backup
    - ../../../../components/ext-auth # if app needs authentication
    - ../../../../components/cnpg/app # if app needs PostgreSQL
  dependsOn:
    - name: postgres-cluster # if using cnpg
      namespace: database
  path: ./kubernetes/apps/self-hosted/my-app/app
  postBuild:
    substitute:
      APP: *app
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
    namespace: flux-system
  targetNamespace: *namespace
```

Key rules:

- Always set `APP: *app` in `postBuild.substitute` when using components
- Use `dependsOn` for anything the app requires at deploy time
- Components are referenced with relative paths from the ks.yaml location (`../../../../components/...`)
- `sourceRef` always points to `flux-system`

### 3. Write the app Kustomization (`app/kustomization.yaml`)

```yaml
---
# yaml-language-server: $schema=https://json.schemastore.org/kustomization
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - helmrelease.yaml
```

### 4. Register the app in the category kustomization

Add `- ./{app-name}/ks.yaml` to the `resources:` list in `kubernetes/apps/{category}/kustomization.yaml`.

### 5. HelmRelease defaults (auto-patched — don't repeat these)

The cluster-level Flux Kustomization automatically patches every HelmRelease with:

- `driftDetection.mode: enabled`
- `install.crds: CreateReplace`
- `rollback: {cleanupOnFail: true, recreate: true}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tholinka/home-ops](https://github.com/tholinka/home-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

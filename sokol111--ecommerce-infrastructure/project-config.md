---
trigger: always_on
description: `ecommerce-infrastructure` is the **deployment + shared-CI repo** for the whole platform. It
---

# AGENTS.md

## What this repo is

`ecommerce-infrastructure` is the **deployment + shared-CI repo** for the whole platform. It
is not a service — it owns no runtime business logic. It holds:

- `helm/` — one thin Helm chart per deployable (5 Go services, 3 Nuxt UIs) plus one shared
  library chart (`shared-helpers`).
- `environments/local/` — the local dev stack: **k3d + Tilt + docker-compose**.
- `environments/production/` — the live **Hetzner k3s** environment: values, raw k8s manifests,
  SOPS secrets, and an ops Makefile driven over an SSH tunnel.
- `docker/` — the Dockerfiles used by **release CI** (`Dockerfile.go`, `.nuxt`, `.seeder`,
  `.logto-seed`). Note there is a *separate* set under `environments/local/docker/` used by
  Tilt for local builds — see **Two sets of Dockerfiles** below.
- `cmd/seeder/` + `cmd/logto-seed/` — two standalone Go programs (each its own module) that
  seed demo data and configure Logto.
- `.github/workflows/` — **reusable** workflows (`workflow_call`) that every service/UI/api repo
  calls. This is the single source of truth for CI/CD across the workspace.

## Component map (this repo)

```
helm/
  shared-helpers/                    library chart (type: library) — all real template logic
  ecommerce-<service>/               thin app chart, depends on shared-helpers via file://
environments/
  local/        Makefile + makefiles/*.mk + Tiltfile + compose/ + values/ + config/
  production/   Makefile + makefiles/*.mk + values/ + k8s/ + grafana/
docker/         release Dockerfiles (used by go-release.yml / nuxt-release.yml / etc.)
cmd/seeder/     demo-data seeder (own go.mod) → ghcr.io/sokol111/ecommerce-seeder
cmd/logto-seed/ Logto bootstrap (own go.mod) → ghcr.io/sokol111/ecommerce-logto-seed
.github/workflows/  reusable CI/CD called by all other repos
```

## Helm: the thin-chart / library-chart pattern

**All template logic lives in `helm/shared-helpers`** (a Helm *library* chart,
`type: library`, version pinned — currently `0.6.0`). Every app chart is deliberately
**thin**: its `templates/*.yaml` are one-line `include` calls into `shared.*` definitions, and
its `values.yaml` supplies the knobs.

```yaml
# helm/ecommerce-catalog-service/templates/deployment.yaml — the whole file
{{ include "shared.deployment" (dict
  "name" (include "template.fullname" .)
  "Labels" (include "template.labels" . | fromYaml)
  ...
) }}
```

Consequences for editing:
- **To change how *all* services are deployed** (probes wiring, config mounting, pod spec,
  labels), edit `helm/shared-helpers/templates/_*.tpl`. This affects every chart at once.
- **To change one service**, edit that chart's `values.yaml` (or the per-env values under
  `environments/*/values/`), not its templates.
- After changing `shared-helpers`, **bump its `version:`** and update the `dependencies` pin
  in each consuming `Chart.yaml`, then `helm dependency update`. Locally, `Tiltfile` runs
  `helm dependency build` automatically (`helm_with_deps`); CI and `make deploy-svc` run
  `helm dependency update` before install.
- Charts vendor the built library under `charts/*.tgz` — this is git-ignored (`.gitignore`),
  rebuilt on demand.

### Values layering
A chart's own `values.yaml` holds safe defaults (image repo, probes, `service.port: 8080`,
ingress disabled). The **real config is layered on per environment**:
- Local: `environments/local/values/<chart>.yaml` (+ `values.debug.yaml` for Go services, which
  wires Delve). The full app config (`mongo`, `kafka`, `security.jwks`, `observability`,
  `tenant.grpc`) is passed as a `config:` YAML block that `shared.deployment` mounts as a
  ConfigMap at `/configs/config.yaml`. Secrets (Logto M2M creds) come via `env` `secretKeyRef`
  into the `logto-credentials` secret.
- Prod: `environments/production/values/<chart>.yaml`.

## Two sets of Dockerfiles (important)

There are two `Dockerfile.go` (and friends) and they are **not** the same:

| Path | Used by | Base build | Notes |
|---|---|---|---|
| `docker/Dockerfile.go` | **release CI** (`go-release.yml`) | pins `go.mod` versions, multi-target `base-release` / `debug-release`, distroless | context = the service repo; `infra` checked out to `infra/` |
| `environments/local/docker/Dockerfile.go` | **Tilt** (local) | reconstructs a `go.work` inside the image | context = workspace root; bundles Delve; ubuntu runtime for debugging |

The local `Dockerfile.go` is the one that implements the "reconstruct `go.work` from local
sources" behavior described in the workspace `AGENTS.md`: Tilt passes `API_DEPS` (a
space-separated list from `GO_SERVICES[...]['api_deps']` in the `Tiltfile`), the Dockerfile
copies each api module out of the build context and `go work use`s it, so **local api changes
flow into the image with no release/tag/bump**. If you add an api dependency to a service,
update its `api_deps` in the `Tiltfile` or the local build will fail with
"API dependency not found".

Release Dockerfiles are consumed by CI via checkout of this repo into `infra/`, so their paths
(`infra/docker/Dockerfile.go`) are load-bearing in `go-release.yml` / `nuxt-release.yml`.

## Local environment (`environments/local/`)

Requires `k3d kubectl tilt helm docker` (`make tools-check`). The Makefile is split into

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sokol111/ecommerce-infrastructure](https://github.com/Sokol111/ecommerce-infrastructure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->

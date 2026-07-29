---
trigger: always_on
description: Helm charts and Kubernetes deployment tools for NATS. The primary chart is `nats` (the NATS server Helm chart); secondary charts include `nack` (JetStream controller), `surveyor`, and legacy/deprecated charts (`nats-operator`, `nats-kafka`, `nats-account-server`).
---

# NATS Kubernetes (k8s)

Helm charts and Kubernetes deployment tools for NATS. The primary chart is `nats` (the NATS server Helm chart); secondary charts include `nack` (JetStream controller), `surveyor`, and legacy/deprecated charts (`nats-operator`, `nats-kafka`, `nats-account-server`).

## Build / Test / Lint Commands

### Lint charts with chart-testing (ct)
```sh
ct lint --all --chart-dirs helm/charts \
  --excluded-charts nats-account-server,nats-kafka,nats-operator,surveyor \
  --validate-maintainers=false
```

### Run Go-based template tests (nats chart)
```sh
cd helm/charts/nats/test && go test
```
Tests use **terratest** with `helm.RenderTemplate` to render templates locally and assert against expected K8s resources. Tests parse rendered NATS config using `nats-server/v2/conf`. Go 1.24+.

### Install tests with chart-testing
```sh
ct install --all --chart-dirs helm/charts \
  --excluded-charts nats-account-server,nats-kafka,nats-operator,surveyor
```
CI creates a microk8s cluster (tested against K8s 1.30, 1.31, 1.32) and applies NACK CRDs before running install tests.

### Render templates locally (for quick validation)
```sh
helm template my-nats helm/charts/nats
helm template my-nats helm/charts/nats -f custom.yaml
```

### Nindent validation (CI check)
CI verifies that `nindent N` values in templates match the actual template indentation level. This applies to `nack` and `nats` charts. Do not use nindent with a value that doesn't match the column position.

## Project Structure

```
helm/
  cr.yaml                          # chart-releaser config (gh-pages branch)
  charts/
    nats/                          # PRIMARY CHART - NATS server
      Chart.yaml                   # apiVersion v2, chart version/appVersion
      values.yaml                  # Canonical reference for all values
      templates/                   # Thin wrappers that call nats.loadMergePatch
      files/                       # Actual resource YAML templates (loaded by .Files.Get)
        config/                    # NATS server config fragments
        stateful-set/              # StatefulSet, pod template, container definitions
        nats-box/                  # NATS Box deployment resources
      test/                        # Go tests (terratest-based)
        chart_test.go              # HelmRender() test harness, Resource types
        defaults_test.go           # Default values assertions
        config_test.go             # Config variations (jetstream, cluster, TLS, etc.)
        ports_test.go              # Port configuration tests
        resources_test.go          # Resource options, merge/patch tests
        tls_test.go                # TLS configuration tests
    nack/                          # NACK - JetStream Controller for K8s
      crds/crds.yml                # CRD definitions (auto-updated by dependabot bump)
    surveyor/                      # NATS Surveyor monitoring
    nats-operator/                 # DEPRECATED: NATS Operator
    nats-kafka/                    # DEPRECATED: NATS-Kafka bridge
    nats-account-server/           # DEPRECATED: Account server
kine-nats/                         # Docker Compose example: kine with NATS backend
```

## Helm Chart Architecture (nats chart)

### Template Pattern: loadMergePatch

The nats chart uses a unique two-layer template architecture:

1. **`templates/` files** are thin wrappers that call `nats.loadMergePatch`
2. **`files/` directory** contains the actual YAML resource definitions, loaded via `.Files.Get`

The `nats.loadMergePatch` helper:
- Loads a YAML file from `files/`
- Runs it through `tpl` (allowing Helm template expressions in files/)
- Deep-merges user-provided `merge:` values
- Applies JSON Patch operations from user-provided `patch:` arrays

This means almost every values.yaml section supports `merge: {}` and `patch: []` for customization.

### Key Helper Templates

| Template | File | Purpose |
|---|---|---|
| `nats.loadMergePatch` | `_helpers.tpl` | Core: load file, merge, patch |
| `nats.defaultValues` | `_helpers.tpl` | Sets computed defaults (resource names, context dirs) |
| `nats.formatConfig` | `_helpers.tpl` | Formats NATS config JSON with `<< >>` unquoting and `$include` directives |
| `tplYaml` | `_tplYaml.tpl` | Processes `$tplYaml` and `$tplYamlSpread` directives in values |
| `jsonpatch` | `_jsonpatch.tpl` | Pure-Helm JSON Patch (RFC 6902) implementation |
| `toPrettyRawJson` | `_toPrettyRawJson.tpl` | JSON output without Go HTML escaping |

### Values Structure Conventions

- **`config.*`** - NATS server configuration (maps to nats.conf sections)
  - Sub-keys: `nats`, `cluster`, `jetstream`, `leafnodes`, `websocket`, `mqtt`, `gateway`, `monitor`, `profiling`, `resolver`
  - Each protocol has `enabled`, `port`, `tls` (with `enabled`, `secretName`, `dir`, `cert`, `key`)
  - Each section has `merge: {}` and `patch: []` for customization
- **`container.*`** - NATS server container spec (image, ports, env, resources)
- **`reloader.*`** - Config reloader sidecar
- **`promExporter.*`** - Prometheus exporter sidecar

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nats-io/k8s](https://github.com/nats-io/k8s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

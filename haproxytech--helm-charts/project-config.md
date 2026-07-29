---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Linting and Template Validation (no cluster needed)

```bash
# Test all charts
./test/local-test.sh

# Test a single chart
./test/local-test.sh haproxy-unified-gateway
./test/local-test.sh kubernetes-ingress

# Helm lint a chart directly
helm lint kubernetes-ingress/
helm lint haproxy-unified-gateway/ -f haproxy-unified-gateway/ci/deployment-default-values.yaml

# Render templates to stdout
helm template test-release kubernetes-ingress/
helm template test-release haproxy-unified-gateway/ --set controller.kind=DaemonSet
helm template test-release haproxy-unified-gateway/ --api-versions monitoring.coreos.com/v1
```

### Chart-testing (ct) — matches GitHub Actions

```bash
# Lint all charts
./test/ct-test.sh lint

# Lint a single chart
./test/ct-test.sh lint haproxy-unified-gateway

# Install (creates a Kind cluster automatically)
./test/ct-test.sh install haproxy-unified-gateway

# Keep the Kind cluster after install tests
KIND_KEEP_CLUSTER=1 ./test/ct-test.sh install
```

### Integration Tests (real Kind cluster)

```bash
# Test all charts
./test/integration-test.sh

# Test a single chart
./test/integration-test.sh haproxy-unified-gateway

# Run a specific test scenario (defaults, daemonset, hpa, pdb, metrics-port, monitoring, hugconf-cleanup, ci)
TEST_FILTER=monitoring ./test/integration-test.sh haproxy-unified-gateway

# Keep namespaces after failure for debugging
KEEP_NS=1 ./test/integration-test.sh
```

## Contribution Requirements

- All commits require a `Signed-off-by` line (DCO): `git commit -s -m "message"`
- Any change to a chart requires a chart **version bump** in `Chart.yaml` following semver
- Submit changes to multiple charts in **separate PRs**

## Project Structure

This repo contains Helm charts for HAProxy products:

- `kubernetes-ingress/` - HAProxy Kubernetes Ingress Controller chart (mature, feature-rich)
- `haproxy/` - HAProxy community chart
- `haproxy-unified-gateway/` - HAProxy Unified Gateway (HUG) chart

## kubernetes-ingress Chart

- **Chart version**: see `kubernetes-ingress/Chart.yaml` (`version` + `appVersion`); follows its own semver
- **Image**: `haproxytech/kubernetes-ingress`
- **Supports**: Deployment + DaemonSet modes, IngressClass, Gateway API, HPA, KEDA, ServiceMonitor/PodMonitor, PDB, proxy service (fetch sync mode), ConfigMap-based HAProxy config, publish-service, default TLS cert generation
- **Templates**: see `kubernetes-ingress/templates/`
- **CI tests**: see `kubernetes-ingress/ci/`
- **Kubernetes**: >=1.23
- **Maintainer**: Dinko Korunic

## haproxy Chart

Community HAProxy chart (not a controller — just runs HAProxy as a workload).
- **Chart**: see `haproxy/Chart.yaml`
- **Kubernetes**: >=1.17 (oldest of the three charts; widest compatibility surface)
- **Templates**: Deployment + DaemonSet, ConfigMap-based config, optional HPA/KEDA, optional Ingress + HTTPRoute, PSP, ServiceMonitor
- **CI tests**: `haproxy/ci/`
- No CRDs, no controller logic — config is purely declarative via `values.yaml` → `configmap.yaml`

## haproxy-unified-gateway Chart

- **Chart version**: see `haproxy-unified-gateway/Chart.yaml` (`version` + `appVersion`)
- **Image**: `haproxytech/haproxy-unified-gateway`
- **Purpose**: Kubernetes Gateway API controller powered by HAProxy
- **Kubernetes**: >=1.26
- **Binary**: `/usr/local/sbin/hug` inside the container
- **Entry point**: `/start.sh`
- **Key flag**: `--hugconf-crd=<namespace>/<name>` for HugConf CRD reference

### Templates

| Template | Purpose |
|---|---|
| `_helpers.tpl` | Name, labels, image, serviceAccount, hugconfCrd, serviceMonitorName, podMonitorName helpers |
| `controller-deployment.yaml` | Deployment (when controller.kind=Deployment) |
| `controller-daemonset.yaml` | DaemonSet with hostNetwork/hostPort support (when controller.kind=DaemonSet) |
| `controller-service.yaml` | NodePort Service (stats port only; HTTP/HTTPS listener ports are added per Gateway by the controller — see PR #354) |
| `controller-serviceaccount.yaml` | ServiceAccount |
| `clusterrole.yaml` | RBAC: Gateway API resources, HUG CRDs (gate.v3.haproxy.org incl. globals/defaults), core K8s resources, auth/authz for kube-rbac metrics |
| `clusterrolebinding.yaml` | ClusterRoleBinding |
| `controller-hugconf.yaml` | HugConf CR (logging, globalRef, defaultsRef configuration) — post-install hook (weight 5) |
| `controller-hugconf-cleanup.yaml` | Pre-delete hook Job that deletes the HugConf CR on `helm uninstall` |
| `controller-crdjob.yaml` | Helm hook Job: installs HUG CRDs (`--job-check-crd`) — post-install hook (weight 0) |
| `controller-crdjob-rbac.yaml` | SA + ClusterRole + Binding for CRD/GWAPI jobs |
| `controller-gwapijob.yaml` | Helm hook Job: installs Gateway API CRDs (`--job-gwapi=VERSION`) |
| `controller-hpa.yaml` | HPA (disabled by default, mutually exclusive with KEDA) |
| `controller-keda.yaml` | KEDA ScaledObject (disabled by default, Deployment only) |
| `controller-servicemonitor.yaml` | ServiceMonitor for Prometheus Operator (disabled by default, gated behind `.Capabilities.APIVersions`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haproxytech/helm-charts](https://github.com/haproxytech/helm-charts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

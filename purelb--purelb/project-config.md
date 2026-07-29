---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PureLB is a Kubernetes service load balancer orchestrator that allocates IP addresses from configured pools and configures Linux networking to announce them. It consists of two main components:

- **Allocator** (`cmd/allocator`): Single cluster-wide pod that watches Services and ServiceGroups, manages IP allocation from pools (local or Netbox IPAM)
- **LBNodeAgent** (`cmd/lbnodeagent`): DaemonSet running on each node, configures local OS networking via netlink to announce allocated IPs

## Importance of IPv6
Always include development of equal IPv6 functionality in every activity.

## Build Commands

All commands via `make`:

```bash
make check          # Run go vet and go test -race -short
make generate       # Generate k8s client code (pkg/generated/)
make crd            # Generate CRD manifests (deployments/crds/)
make image          # Build container images using ko (see below for deployment)
make manifest       # Generate k8s deployment YAML via kustomize
make helm           # Package Helm chart
make scan           # Run govulncheck for vulnerabilities
make run-allocator  # Run allocator locally (needs kubeconfig)
make run-lbnodeagent # Run node agent locally (set PURELB_NODE_NAME)
```

Run a single test:
```bash
go test -race -run TestName ./internal/allocator/...
```

For tests requiring Netbox integration, set `NETBOX_BASE_URL` and `NETBOX_USER_TOKEN` environment variables.

## Building and Deploying to Test Cluster

**IMPORTANT**: The default `make image` builds to `ko.local/` which requires local Docker. For deploying to the test cluster, you must use `ko` directly with the correct registry and tag.

### There are multiple custers used for texting. Check the cluster in use
```bash
kubectx
```

### Check Current Cluster Image Tags

First, check what image tags the cluster is currently using:
```bash
kubectl get daemonset lbnodeagent -n purelb-system-o jsonpath='{.spec.template.spec.containers[0].image}'
# Example output: ghcr.io/purelb/purelb/lbnodeagent:general_k8_update
```

### Build and Push with ko

Use `ko` directly with the correct registry (`ghcr.io/purelb/purelb`) and tag (matching the current branch/deployment):
```bash
# Set the registry and TAG (both required - TAG is used by .ko.yaml for ldflags)
export KO_DOCKER_REPO=ghcr.io/purelb/purelb
export TAG=general_k8_update  # Must match the tag you're deploying

# Build and push with the correct tag (match current cluster deployment)
go run github.com/google/ko@v0.17.1 build --base-import-paths --tags=$TAG ./cmd/lbnodeagent
go run github.com/google/ko@v0.17.1 build --base-import-paths --tags=$TAG ./cmd/allocator
```

### Restart Pods to Pick Up New Images

After pushing new images, restart the pods to pull the updated images:
```bash
kubectl rollout restart daemonset/lbnodeagent -n purelb-system
kubectl rollout restart deployment/allocator -n purelb-system

# Wait for rollout to complete
kubectl rollout status daemonset/lbnodeagent -n purelb-system
kubectl rollout status deployment/allocator -n purelb-system
```

### Common Mistakes to Avoid

1. **Don't use `make image`** for cluster deployment - it builds to `ko.local/` which requires local Docker daemon
2. **Always check the current image tag** before building - use the same tag the cluster expects
3. **Remember to restart pods** after pushing - Kubernetes won't automatically pull updated images with the same tag

## Architecture

```
┌──────────────────┐         ┌─────────────────────┐
│ Allocator        │         │ LBNodeAgent (per node)│
│ - IP allocation  │         │ - netlink config    │
│ - Pool mgmt      │         │ - Election leader   │
└────────┬─────────┘         └──────────┬──────────┘
         │                              │
         └──────────┬───────────────────┘
                    │
          K8s API Server
                    │
    ┌───────────────┴───────────────┐
    │ CRDs: ServiceGroup, LBNodeAgent│
    └───────────────────────────────┘
```

## Key Internal Packages

- `internal/allocator/` - IP pool management and service allocation logic. Supports LocalPool (in-memory) and NetboxPool (external IPAM)
- `internal/local/` - Linux networking via netlink (interfaces, routes, ARP/NDP). Contains `LocalAnnouncer` implementation
- `internal/k8s/` - Kubernetes client integration using informers and work queues. The `Client` struct watches Services/Endpoints and invokes callbacks on changes
- `internal/election/` - Lease-based subnet-aware leader election for node agents. Each node maintains a Kubernetes Lease with its subnets annotated; uses SHA256 hash of (node name + service key) to deterministically elect a winner from nodes that have the IP's subnet

## Custom Resources

Defined in `pkg/apis/purelb/v1/`:

- **ServiceGroup**: Defines IP pools (local CIDR ranges or Netbox references), supports dual-stack IPv4/IPv6
- **LBNodeAgent**: Node-specific announcement configuration (interface selection, gratuitous ARP)

Key annotations in `pkg/apis/purelb/v1/annotations.go`:
- `purelb.io/service-group` - User sets to request allocation from specific pool

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [purelb/purelb](https://github.com/purelb/purelb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

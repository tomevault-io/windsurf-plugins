---
trigger: always_on
description: This document provides step-by-step instructions for building, deploying, and testing the Argo Rollouts AI Metric Plugin.
---

# Agent Instructions for Argo Rollouts AI Metric Plugin

This document provides step-by-step instructions for building, deploying, and testing the Argo Rollouts AI Metric Plugin.

## Prerequisites

- Docker installed and running
- Kind (Kubernetes in Docker) cluster running
- kubectl configured to access the Kind cluster
- Go 1.24.0+ installed (toolchain 1.24.3 recommended)
- GNU Make installed

## Available Make Targets

View all available Make targets:

```bash
# Show all available targets with descriptions
make help
```

Common targets:
- `make build` - Build the Go binary
- `make test` - Run unit tests
- `make docker-build` - Build Docker image
- `make docker-push` - Push Docker image to registry
- `make docker-buildx` - Build and push multi-platform image
- `make fmt` - Format Go code
- `make vet` - Run Go vet
- `make lint` - Run linter

## Git Workflow

All commits must be signed off to certify the [Developer Certificate of Origin (DCO)](https://developercertificate.org/). Use the `-s` flag when committing:

```bash
git commit -s -m "feat: my change"
```

This adds a `Signed-off-by: Name <email>` trailer to the commit message. PRs with unsigned commits will fail the DCO check.

## 1. Build the Plugin Image

Build the Docker image for the plugin using Make:

```bash
# Build image with default settings (uses IMG from Makefile: csanchez/rollouts-plugin-metric-ai:latest)
make docker-build

# Build for multiple platforms (ARM64, AMD64, etc.) - pushes to registry
make docker-buildx

# Build for specific platforms only
make docker-buildx PLATFORMS=linux/arm64,linux/amd64
```

**Note:** 
- The `docker-buildx` target will push to a registry. For local Kind usage, use `docker-build` instead.
- Default image name is defined in Makefile: `csanchez/rollouts-plugin-metric-ai:latest`

## 2. Setup Kind Cluster

Create a Kind cluster for testing (if it doesn't exist):

```bash
# Create Kind cluster with default name
kind create cluster --name rollouts-plugin-metric-ai-test-e2e
```

## 3. Load Image into Kind Cluster

Load the built image into your Kind cluster:

```bash
# Load the image into Kind using the default cluster name
kind load docker-image csanchez/rollouts-plugin-metric-ai:latest --name rollouts-plugin-metric-ai-test-e2e

# Verify the image was loaded
docker exec -it rollouts-plugin-metric-ai-test-e2e-control-plane crictl images | grep rollouts-plugin-metric-ai
```

**Note:** The default Kind cluster name from Makefile is `rollouts-plugin-metric-ai-test-e2e`

## 5. Deploy Argo Rollouts with Plugin

Deploy Argo Rollouts with the AI metric plugin using Kustomize:

```bash
# Deploy Argo Rollouts with plugin configuration
kubectl apply -n argo-rollouts -k config/argo-rollouts

# Wait for the deployment to be ready
kubectl rollout status deployment/argo-rollouts -n argo-rollouts --timeout=5m

# Verify the controller is running
kubectl get pods -n argo-rollouts
```

## 6. Update Plugin After Code Changes

After making changes and rebuilding, restart the controller:

```bash
# Restart the Argo Rollouts controller
kubectl rollout restart deployment/argo-rollouts -n argo-rollouts

# Wait for the rollout to complete
kubectl rollout status deployment/argo-rollouts -n argo-rollouts --timeout=5m

# Verify the controller is running
kubectl get pods -n argo-rollouts
```

## 7. Get Argo Rollouts Controller Logs

View the logs to debug and monitor the plugin:

```bash
# Follow logs in real-time
kubectl logs -f -n argo-rollouts deployment/argo-rollouts

# Or get recent logs
kubectl logs -n argo-rollouts deployment/argo-rollouts --tail=100

# Filter for plugin-specific logs
kubectl logs -n argo-rollouts deployment/argo-rollouts | grep -i "metric-ai\|plugin\|quota"

# View logs with timestamps
kubectl logs -n argo-rollouts deployment/argo-rollouts --timestamps=true
```

## 8. Deploy Test Application

Deploy a sample application for canary testing:

```bash
# Apply the canary demo resources using Kustomize
kubectl apply -k config/rollouts-examples/

# This deploys:
# - Canary service (stable and preview)
# - Rollout with AI analysis
# - Analysis template
# - Traffic generator (hits /color endpoint every second)
```

### Monitor Traffic Generator

The traffic generator continuously hits both stable and canary services:

```bash
# View traffic generator logs
kubectl logs -f deployment/traffic-generator -n rollouts-test-system

# Example output:
# [2025-10-01 12:00:00] STABLE: blue (HTTP 200)
# [2025-10-01 12:00:00] CANARY: yellow (HTTP 200)
```

## 9. Trigger a Canary Rollout

### Option A: Update the Rollout Image

Trigger a canary deployment by updating the image:

```bash
# Update the rollout to a new image version
kubectl argo rollouts set image canary-demo \
  canary-demo=argoproj/rollouts-demo:yellow \
  -n rollouts-test-system

# Or manually edit the rollout
kubectl edit rollout canary-demo -n rollouts-test-system
```

### Option B: Use kubectl patch

```bash
# Patch the rollout with a new image
kubectl patch rollout canary-demo -n rollouts-test-system --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"argoproj/rollouts-demo:yellow"}]'
```

### Option C: Restart the Rollout

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [argoproj-labs/rollouts-plugin-metric-ai](https://github.com/argoproj-labs/rollouts-plugin-metric-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->

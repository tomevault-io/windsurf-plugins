---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go project for managing Slicer VMs using the Slicer SDK. The project provides automation tooling to recreate environments from scratch and create VMs with presets. A Magefile is used to manage VM lifecycle operations.

**Documentation reference**: The `docs.slicervm.com/` directory contains the official Slicer documentation (cloned locally). Key references:
- `docs/examples/buildkit.md` - BuildKit VM deployment pattern
- `docs/reference/api.md` - REST API endpoints
- `docs/tasks/userdata.md` - VM bootstrap scripts

## Build Commands

```bash
# List all mage targets
mage -l

# BuildKit targets
mage buildkit:deploy              # Create a new BuildKit VM
mage buildkit:list                # List all BuildKit VMs
mage buildkit:delete <hostname>   # Delete a BuildKit VM
mage buildkit:logs <hostname>     # Show serial console logs
mage buildkit:userdata            # Print the userdata script
mage buildkit:yaml                # Generate Slicer config YAML

# OpenFaaS Edge targets
mage openfaas:deploy              # Create a new OpenFaaS Edge VM
mage openfaas:list                # List all OpenFaaS VMs
mage openfaas:delete <hostname>   # Delete an OpenFaaS VM
mage openfaas:logs <hostname>     # Show serial console logs
mage openfaas:userdata            # Print the userdata script
mage openfaas:yaml                # Generate Slicer config YAML

# RustFS (S3-compatible storage) targets
mage rustfs:deploy                # Create a new RustFS VM
mage rustfs:list                  # List all RustFS VMs
mage rustfs:delete <hostname>     # Delete a RustFS VM
mage rustfs:logs <hostname>       # Show serial console logs
mage rustfs:userdata              # Print the userdata script

# PostgreSQL targets
mage postgres:deploy              # Create a new PostgreSQL VM (Gitea-ready)
mage postgres:list                # List all PostgreSQL VMs
mage postgres:delete <hostname>   # Delete a PostgreSQL VM
mage postgres:logs <hostname>     # Show serial console logs
mage postgres:userdata            # Print the userdata script
mage postgres:yaml                # Generate Slicer config YAML

# Gitea targets
mage gitea:deploy                 # Create a new Gitea VM (requires postgres, rustfs)
mage gitea:list                   # List all Gitea VMs
mage gitea:delete <hostname>      # Delete a Gitea VM
mage gitea:logs <hostname>        # Show serial console logs
mage gitea:userdata               # Print the userdata script
mage gitea:yaml                   # Generate Slicer config YAML

# Gitea Runner targets
mage runner:deploy                # Create a new Gitea Actions Runner VM
mage runner:list                  # List all Runner VMs
mage runner:delete <hostname>     # Delete a Runner VM
mage runner:logs <hostname>       # Show serial console logs
mage runner:userdata              # Print the userdata script
mage runner:yaml                  # Generate Slicer config YAML

# K3s Kubernetes cluster targets
mage k3s:deploy                   # Create a new K3s cluster (control plane + workers)
mage k3s:list                     # List all K3s VMs
mage k3s:deleteCluster            # Delete entire K3s cluster
mage k3s:logsCP                   # Show control plane logs
mage k3s:scaleWorkers <count>     # Scale worker nodes

# Crossplane targets
mage crossplane:install           # Install Crossplane in K3s cluster
mage crossplane:uninstall         # Uninstall Crossplane

# Grafana stack targets (Prometheus, Grafana, Alertmanager)
mage grafana:install              # Install kube-prometheus-stack in K3s cluster
mage grafana:uninstall            # Uninstall Grafana stack
mage grafana:status               # Show deployment/pod status
mage grafana:services             # Show service endpoints and NodePorts
mage grafana:password             # Get Grafana admin password
mage grafana:logs [pod-name]      # Show logs from a monitoring pod

# cert-manager targets (TLS certificate automation)
mage certManager:install          # Install cert-manager in K3s cluster
mage certManager:uninstall        # Uninstall cert-manager
mage certManager:status           # Show deployment/pod status
mage certManager:clusterIssuer    # Create Let's Encrypt ClusterIssuer (requires ACME_EMAIL)
mage certManager:clusterIssuerList # List all ClusterIssuers
mage certManager:logs [pod-name]  # Show logs from a cert-manager pod

# Run tests
go test ./...

# Run a single test
go test -run TestName ./path/to/package
```

## Architecture

### Slicer SDK Usage

The project uses `github.com/slicervm/sdk` for programmatic VM management:

```go
import sdk "github.com/slicervm/sdk"

// Create client
client := sdk.NewSlicerClient(baseURL, token, "user-agent", nil)

// Create VM in a host group
client.CreateNode(ctx, "hostgroup-name", sdk.SlicerCreateNodeRequest{
    RamGB:    4,
    CPUs:     2,
    Userdata: "#!/bin/bash\n...",
})

// Delete VM
client.DeleteVM(ctx, "hostgroup-name", "hostname")
```

### Package Structure

- `pkg/buildkit/` - BuildKit VM deployment with embedded userdata script
- `pkg/openfaas/` - OpenFaaS Edge VM deployment with embedded userdata script

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gaarutyunov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

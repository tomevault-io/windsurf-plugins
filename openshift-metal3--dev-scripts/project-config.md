---
trigger: always_on
description: Generates the `install-config.yaml`:
---

# OpenShift Dev Scripts Guide

## Introduction

The `dev-scripts` repository provides a comprehensive set of scripts and automation for deploying OpenShift on baremetal infrastructure, with a focus on development, testing, and CI/CD workflows. It supports multiple deployment methods including traditional IPI (Installer Provisioned Infrastructure) using Ironic, and Agent-based installation.

The project is primarily designed for:
- **Development**: Testing OpenShift features and components locally
- **CI/CD**: Automated testing in OpenShift CI pipelines
- **Baremetal Testing**: Simulating baremetal deployments using libvirt VMs
- **Edge Scenarios**: Testing Single Node OpenShift (SNO) and compact clusters

## Repository Structure

```
dev-scripts/
├── 01_install_requirements.sh       # Install system dependencies
├── 02_configure_host.sh             # Configure host networking and VMs
├── 03_build_installer.sh            # Extract openshift-install binary
├── 04_setup_ironic.sh               # Setup Ironic containers
├── 05_create_install_config.sh      # Generate install-config.yaml
├── 06_create_cluster.sh             # Deploy the cluster
│
├── agent/                           # Agent-based installation scripts
│   ├── 01_agent_requirements.sh
│   ├── 03_agent_build_installer.sh
│   ├── 04_agent_prepare_release.sh
│   ├── 05_agent_configure.sh
│   ├── 06_agent_create_cluster.sh
│   ├── 07_agent_add_extraworker_nodes.sh
│   └── docs/                        # Agent-specific documentation
│
├── assets/                          # Configuration templates and patches
├── metal3-dev/                      # Metal3 development utilities
├── metallb/                         # MetalLB deployment scripts
├── network-configs/                 # Network configuration examples
│   ├── bond/                        # Bonded network configs
│   ├── static/                      # Static IP configs
│   └── nmstate-brex-bond/           # NMState configs
│
├── docs/                            # Additional documentation
├── config_example.sh                # Comprehensive configuration examples
├── common.sh                        # Shared functions and utilities
├── Makefile                         # Main automation entry point
└── README.md                        # Primary documentation
```

## General Workflow

The deployment follows a sequential workflow through six main steps:

### Step 01: Install Requirements
**Script**: `01_install_requirements.sh`

Installs all prerequisite packages and tools:
- System packages (libvirt, podman, ansible, etc.)
- OpenShift client tools (`oc`, `kubectl`)
- Development tools (Go, yq, jq)
- Starts local container registry (if `ENABLE_LOCAL_REGISTRY=true`)

**Key Environment Variables**:
- `ANSIBLE_VERSION`: Override Ansible version
- `GO_VERSION`: Override Go version
- `OPENSHIFT_CLIENT_TOOLS_URL`: Custom oc/kubectl download URL

### Step 02: Configure Host
**Script**: `02_configure_host.sh`

Configures the host environment:
- Creates libvirt networks (`baremetal`, `provisioning`)
- Configures firewall rules and NAT
- Creates virtual BMC nodes for testing
- Sets up networking bridges
- Generates SSH keys if needed
- Configures NTP/chronyd

**Key Environment Variables**:
- `WORKING_DIR`: Base directory for all operations (default: `/opt/dev-scripts`)
- `CLUSTER_NAME`: OpenShift cluster name (default: `ostest`)
- `NUM_MASTERS`: Number of control plane nodes (default: `3`)
- `NUM_WORKERS`: Number of worker nodes (default: `2`)

### Step 03: Build Installer
**Script**: `03_build_installer.sh`

Extracts the `openshift-install` binary from the release payload:
- Downloads the specified OCP release image
- Extracts `openshift-install` tool
- Caches the installer for reuse

**Key Environment Variables**:
- `OPENSHIFT_RELEASE_IMAGE`: Specific release image to deploy
- `OPENSHIFT_RELEASE_STREAM`: Release stream (e.g., `4.21`)
- `OPENSHIFT_RELEASE_TYPE`: Release type (`nightly`, `ci`, `ga`, `okd`)

### Step 04: Setup Ironic
**Script**: `04_setup_ironic.sh`

Deploys Ironic baremetal provisioning services:
- Starts Ironic containers (ironic, ironic-inspector, httpd, mariadb)
- Configures DHCP and TFTP services
- Sets up image caching
- Prepares provisioning network

**Note**: This step is only used for traditional IPI installations, not for Agent-based installations.

### Step 05: Create Install Config
**Script**: `05_create_install_config.sh`

Generates the `install-config.yaml`:
- Configures cluster networking (IPv4/IPv6/dual-stack)
- Sets up node definitions and BMC credentials
- Configures pull secrets and SSH keys
- Applies customizations from environment variables

**Key Environment Variables**:
- `IP_STACK`: IP configuration (`v4`, `v6`, `v4v6`, `v6v4`)
- `BASE_DOMAIN`: Cluster base domain
- `FIPS_MODE`: Enable FIPS compliance
- `NETWORK_TYPE`: CNI plugin (`OVNKubernetes`, `OpenShiftSDN`)

### Step 06: Create Cluster
**Script**: `06_create_cluster.sh`

Deploys the OpenShift cluster:
- Runs `openshift-install create cluster`
- Monitors installation progress
- Generates `kubeconfig` and `clouds.yaml`
- Performs post-installation validation
- Creates cluster credentials

**Key Environment Variables**:
- `OPENSHIFT_INSTALL_RELEASE_IMAGE_OVERRIDE`: Override release image

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift-metal3/dev-scripts](https://github.com/openshift-metal3/dev-scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an Ansible automation framework for Telco Verification CI/CD pipelines. It provides end-to-end OpenShift cluster deployment, CNF (Cloud-Native Network Function) testing, and infrastructure management capabilities for OpenShift Edge computing deployments.

## Common Commands

### Install Dependencies
```bash
# Install Ansible collection dependencies
ansible-galaxy collection install -r requirements.yml
```

### Running Playbooks

**Deploy OpenShift Hybrid Multinode Cluster:**
```bash
# Deploy latest version of a specific minor release
ansible-playbook ./playbooks/deploy-ocp-hybrid-multinode.yml \
  -i ./inventories/ocp-deployment/build-inventory.py \
  --extra-vars 'release=4.17'

# Deploy specific version
ansible-playbook ./playbooks/deploy-ocp-hybrid-multinode.yml \
  -i ./inventories/ocp-deployment/build-inventory.py \
  --extra-vars 'release=4.17.9'

# Deploy with trusted internal registry (disconnected mode)
ansible-playbook ./playbooks/deploy-ocp-hybrid-multinode.yml \
  -i ./inventories/ocp-deployment/build-inventory.py \
  --extra-vars "release=4.17.9 internal_registry=true"
```

**Deploy OpenShift Operators:**
```bash
# Connected mode
ansible-playbook ./playbooks/deploy-ocp-operators.yml \
  -i ./inventories/ocp-deployment/build-inventory.py \
  --extra-vars 'kubeconfig="/path/to/kubeconfig" version="4.17" operators=[...]'

# Disconnected mode with internal registry mirroring
ansible-playbook ./playbooks/deploy-ocp-operators.yml \
  -i ./inventories/ocp-deployment/build-inventory.py \
  --extra-vars 'kubeconfig="/path/to/kubeconfig" disconnected=true version="4.17" operators=[...]'
```

**Setup Cluster Environment:**
```bash
ansible-playbook ./playbooks/setup-cluster-env.yml --extra-vars 'release=4.17'
```

### Linting
```bash
# Ansible linting
ansible-lint

# YAML linting
yamllint playbooks/
```

### Container Image Build
```bash
# Build container image
podman build -f Containerfile -t eco-ci-cd:latest .
```

### Running Tests
```bash
# Run Chainsaw DAST tests
chainsaw test tests/dast/
```

## Architecture Overview

### Inventory Management
The repository uses a **dynamic inventory system** centered around `inventories/ocp-deployment/build-inventory.py`. Inventory data is stored in structured directories:
- `inventories/ocp-deployment/group_vars/` - Group-level variables
- `inventories/ocp-deployment/host_vars/` - Host-specific variables (bastion, workers, masters, hypervisors)
- `inventories/cnf/` - CNF testing inventories
- `inventories/infra/` - Infrastructure inventories

### Deployment Workflow Pattern
OpenShift deployments follow an agent-based installation pattern with several key phases:

1. **Environment Preparation** (Bastion Setup)
   - Install dependencies and extract OpenShift installer
   - Use `ocp_version_facts` role to retrieve release information
   - Configure HTTP storage for artifacts

2. **Manifest Generation**
   - Generate installation manifests using `redhatci.ocp.generate_manifests`
   - Support for additional custom manifests via `extra_manifests` variable
   - Generate agent ISO with `redhatci.ocp.generate_agent_iso`

3. **Virtual Infrastructure Setup**
   - Setup sushy tools for out-of-band interface emulation on KVM hosts
   - Deploy VMs on hypervisors using `redhatci.ocp.create_vms`
   - Process KVM nodes to set proper facts

4. **Node Provisioning and Booting**
   - Boot bare-metal workers and virtual masters using `redhatci.ocp.boot_iso`
   - Monitor installation with `redhatci.ocp.monitor_agent_based_installer`

5. **Post-Installation Configuration**
   - Configure cluster pull secrets for additional trusted registries
   - Optional: Setup internal registry with DNS and CA trust (when `internal_registry=true`)

### Disconnected/Air-Gapped Pattern
When `internal_registry=true` or `disconnected=true`:
- Bastion hosts internal container registry on port 5000
- DNS (dnsmasq) configured to resolve registry URL to bastion IP
- Registry CA certificate added to cluster trust via ConfigMap
- Pull secrets updated with registry credentials
- Operators mirrored using `ocp_operator_mirror` role before installation

### Network Configuration
Nodes support **dynamic network interface configuration** via environment variables:
- `<inventory_hostname>_EXTERNAL_INTERFACE` - Network interface name (e.g., `worker0_EXTERNAL_INTERFACE=eth2`)
- `<inventory_hostname>_MAC_ADDRESS` - MAC address (e.g., `worker0_MAC_ADDRESS=aa:bb:cc:aa:bb:cc`)

This allows flexible network setup without modifying inventory files directly.

### Operator Deployment Pattern
The `deploy-ocp-operators.yml` playbook supports both connected and disconnected flows:
- **Connected:** Direct installation from Red Hat catalogs
- **Disconnected:** Mirror operators to internal registry, generate ImageDigestMirrorSets (IDMS) and CatalogSources, then install

Operators are defined as a list with fields: `name`, `catalog`, `nsname`, `channel`, `og_name`, `deploy_default_config`.

### Roles Architecture

**Core Infrastructure Roles** (in `playbooks/roles/`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openshift-kni) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

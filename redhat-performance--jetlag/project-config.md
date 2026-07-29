---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Jetlag is an OpenShift cluster deployment tool that uses Ansible automation to deploy Multi Node OpenShift (MNO) and Single Node OpenShift (SNO) clusters via the Assisted Installer. It supports Red Hat performance labs, Scale Labs, and IBMcloud environments.

## Essential Commands

### Environment Setup
```bash
# Bootstrap ansible virtual environment (run from repo root)
source bootstrap.sh

# Red Hat Labs (Scale Lab/Performance Lab)
# Copy and edit configuration file
cp ansible/vars/all.sample.yml ansible/vars/all.yml
# Edit all.yml with your lab configuration (lab, lab_cloud, cluster_type, etc.)

# Create inventory file for your lab environment
ansible-playbook ansible/create-inventory.yml
# Setup bastion host (replace cloud99.local with your inventory file)
ansible-playbook -i ansible/inventory/cloud99.local ansible/setup-bastion.yml

# IBMcloud
# Copy and edit configuration file
cp ansible/vars/ibmcloud.sample.yml ansible/vars/ibmcloud.yml
# Edit ibmcloud.yml with your IBMcloud configuration (cluster_type, worker_node_count, etc.)

# Create inventory file from IBMcloud CLI data
ansible-playbook ansible/ibmcloud-create-inventory.yml
# Setup bastion host for IBMcloud
ansible-playbook -i ansible/inventory/ibmcloud.local ansible/ibmcloud-setup-bastion.yml
```

### Cluster Deployment
```bash
# Red Hat Labs (Scale Lab/Performance Lab)
# Deploy Multi Node OpenShift cluster
ansible-playbook -i ansible/inventory/cloud99.local ansible/mno-deploy.yml

# Deploy Single Node OpenShift clusters
ansible-playbook -i ansible/inventory/cloud99.local ansible/sno-deploy.yml

# Deploy Virtual Multi Node OpenShift (VMNO) - requires hypervisor setup first
ansible-playbook -i ansible/inventory/cloud99.local ansible/hv-setup.yml
ansible-playbook -i ansible/inventory/cloud99.local ansible/hv-vm-create.yml
ansible-playbook -i ansible/inventory/cloud99.local ansible/mno-deploy.yml

# IBMcloud
# Deploy Multi Node OpenShift on IBMcloud
ansible-playbook -i ansible/inventory/ibmcloud.local ansible/ibmcloud-mno-deploy.yml

# Deploy Single Node OpenShift on IBMcloud
ansible-playbook -i ansible/inventory/ibmcloud.local ansible/ibmcloud-sno-deploy.yml
```

### Cluster Management
```bash
# Scale out MNO cluster
ansible-playbook ansible/ocp-scale-out.yml

# Setup hypervisor nodes for VMs
ansible-playbook ansible/hv-setup.yml

# Create VMs on hypervisor nodes
ansible-playbook ansible/hv-vm-create.yml

# Delete VMs from hypervisor nodes
ansible-playbook ansible/hv-vm-delete.yml

# Replace VMs on hypervisor nodes (delete + recreate)
ansible-playbook ansible/hv-vm-replace.yml

# Sync OpenShift releases
ansible-playbook ansible/sync-ocp-release.yml

# Deploy MinIO object storage on the bastion (set setup_bastion_minio: true in all.yml first)
ansible-playbook -i ansible/inventory/cloud99.local ansible/bastion-minio.yml

# Clean all MinIO data between cluster deployments (wipes data, recreates empty buckets)
ansible-playbook -i ansible/inventory/cloud99.local ansible/bastion-minio-clean.yml
```

## Project Architecture

### Key Configuration Files
- `ansible/vars/all.yml` - Main configuration for Red Hat labs (copy from `ansible/vars/all.sample.yml`)
- `ansible/vars/ibmcloud.yml` - IBMcloud-specific configuration (copy from `ansible/vars/ibmcloud.sample.yml`)
- `pull-secret.txt` - OpenShift pull secret (place in repo root)
- `ansible/inventory/$CLOUDNAME.local` - Generated inventory file for your lab

### Critical Variables
- `lab`: Environment type (`performancelab`, `scalelab`, or `ibmcloud`)
- `lab_cloud`: Specific cloud allocation (e.g., `cloud42`)
- `cluster_type`: Either `mno`, `sno`, or `vmno`
- `worker_node_count`: Number of bare metal worker nodes for MNO clusters
- `hybrid_worker_count`: Number of virtual worker nodes for hybrid MNO clusters (requires hypervisor setup)
- `ocp_build`: OpenShift build type (`ga`, `dev`, or `ci`)
- `ocp_version`: OpenShift version (e.g., `latest-4.20`)

### Ansible Role Structure
Jetlag uses a modular Ansible role architecture:

- **Bastion roles**: `bastion-*` roles configure the bastion host with services like Assisted Installer, DNS, registry
- **Installation roles**: `install-cluster`, `sno-post-cluster-install` handle cluster deployment
- **Hypervisor roles**: `hv-*` roles manage VM infrastructure on hypervisor nodes
- **Utility roles**: `boot-iso`, `sync-*` roles provide supporting functionality

### Cluster Types
- **MNO (Multi Node OpenShift)**: 3 control-plane nodes + configurable bare metal worker nodes
- **SNO (Single Node OpenShift)**: Single node clusters, one per available machine
- **VMNO (Virtual Multi Node OpenShift)**: MNO cluster using VMs instead of bare metal (Jetlag-specific term)
- **Hybrid MNO**: MNO cluster with both bare metal and virtual worker nodes

#### Virtual and Hybrid Cluster Details
- **VMNO clusters** allow multi-node deployment with fewer physical machines (minimum: 1 bastion + 1-2 hypervisors)
- **Hybrid clusters** combine bare metal workers (`worker_node_count`) with virtual workers (`hybrid_worker_count`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redhat-performance/jetlag](https://github.com/redhat-performance/jetlag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

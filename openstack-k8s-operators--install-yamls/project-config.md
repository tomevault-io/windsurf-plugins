---
trigger: always_on
description: install_yamls is a k8s YAML generator and installer for Cloud Native OpenStack
---

# AGENTS.md

## What is this repository

install_yamls is a k8s YAML generator and installer for Cloud Native OpenStack
(RHOSO -- Red Hat OpenStack Services on OpenShift). It automates deploying
OpenStack operators and services on a **pre-installed** OpenShift cluster.

The upstream repository lives at
`https://github.com/openstack-k8s-operators/install_yamls`.

## Tech stack

- **GNU Make** as the primary user interface (two Makefiles).
- **Bash** scripts for YAML generation (kustomize), operator lifecycle, and
  environment setup.
- **Ansible** playbooks and roles under `devsetup/` for tool download and
  NFS provisioning.
- **kustomize** for CR/resource generation (output to `out/`).
- **Zuul** for CI (end-to-end jobs, no unit test suite).

## Two-layer architecture

The repository has an intentional two-layer split. Agents **must** respect
this boundary.

| Layer | Entry point | Scope | Sudo required |
|---|---|---|---|
| Root | `Makefile` + `scripts/` | Generate YAML, run `oc`/`kubectl` against an existing OCP cluster. Must NOT modify the local machine. | **No** |
| devsetup | `devsetup/Makefile` + `devsetup/scripts/` | Bootstrap the dev environment: CRC install, libvirt VMs for EDPM nodes, BMaaS LAB, TripleO standalone. CAN modify the host. | **Yes** |

## Repository layout

| Path | Description |
|---|---|
| `Makefile` | Primary user API (~2900 lines). All operator deploy/cleanup targets. |
| `scripts/` | 55+ shell scripts: kustomize generators (`gen-*`), operator OLM lifecycle (`operator-*`), storage helpers, mirror registry, cleanup. |
| `devsetup/` | CRC automation, EDPM VM creation, BMaaS LAB, standalone OSP 17.1, tool download. Has its own `Makefile`. |
| `devsetup/scripts/` | Host-level automation scripts (CRC setup, EDPM node creation, networking). |
| `devsetup/roles/` | Ansible roles: `download_tools`, `nfs_server`. |
| `devsetup/standalone/` | TripleO standalone templates and scripts (used for adoption testing). |
| `crc/` | Kustomize overlays for CRC local storage. |
| `zuul.d/` | Zuul CI job definitions (`jobs.yaml`, `projects.yaml`). |
| `out/` | Generated output directory (gitignored). |
| `OWNERS` | Prow/K8s-style approvers and reviewers list. |

## Critical rules

### No host mutation from root Makefile

The root `Makefile` and `scripts/` must **never** require sudo or change
the local machine beyond what `oc`/`kubectl` do. If a new feature needs
host-level changes, it belongs in `devsetup/`.

### Output directory

Generated YAML goes to `${OUT}` (defaults to `./out/`). This directory is
gitignored. Never commit its contents.

### Variable overrides

All Makefile variables use `?=` (conditional assignment). Override via
environment or command-line:

```
NETWORK_ISOLATION=false make openstack_deploy
```

### Do not commit secrets

The following are gitignored and must never be committed:
`pull-secret.txt`, `kubeconfig`, `ansibleee-ssh-key-id_rsa*`, `tls.crt`.

### Script naming conventions

| Pattern | Purpose |
|---|---|
| `gen-*.sh` | Kustomize/CR generators |
| `operator-*.sh` | Operator OLM lifecycle (deploy, wait, cleanup) |
| `openstack-*.sh` | OpenStack CRD and update helpers |
| `cleanup-*.sh` | Resource removal |
| `mirror-registry-*.sh` | Disconnected environment helpers |
| `storage_*.sh` | CRC PV storage management |

### OKD support

Pass `OKD=true` to any root Makefile target to deploy on OKD community
distro instead of OCP.

## Key Makefile targets (root)

| Phase | Targets | Description |
|---|---|---|
| Prerequisites | `crc_storage`, `namespace`, `operator_namespace`, `input` | Storage PVs, namespace creation, secrets/configmaps |
| Operator install | `openstack`, `keystone`, `mariadb`, `neutron`, ... | Install operators via OLM |
| Service deploy | `openstack_deploy`, `openstack_init`, `*_deploy` | Deploy OpenStack service CRs |
| EDPM | `edpm_deploy`, `edpm_wait_deploy`, `edpm_nova_discover_hosts` | Data plane deployment |
| Update | `update_services`, `update_system` | Rolling update workflow |
| Cleanup | `cleanup`, `deploy_cleanup`, `*_cleanup` | Remove operators/services |
| Mirror registry | `mirror_registry`, `mirror_registry_secure`, `mirror_registry_cleanup` | Disconnected environment testing |
| Utility | `wait`, `help` | Operator readiness check, target listing |

## Key Makefile targets (devsetup)

| Target | Description |
|---|---|
| `crc` | Install and configure CRC (Code Ready Containers) |
| `download_tools` | Install all required development tools |
| `edpm_compute` | Create EDPM compute node VMs via libvirt |
| `edpm_baremetal_compute` | Create virtual baremetal VMs with sushy-tools (Redfish BMC) |
| `crc_attach_default_interface` | Attach libvirt default network to CRC VM |
| `bmaas` | Create BMaaS (Bare Metal as a Service) LAB |
| `edpm_deploy_instance` | Quick validation: deploy a test Nova instance |
| `ipv6_lab` | IPv6 NAT64 lab setup with SNO |
| `network_isolation_bridge` | Create isolated network bridge |

## Operator and architecture data flow

`make *_repo` targets (for example `make openstack_repo`) clone individual
operator repositories into `${OPERATOR_BASE_DIR}` (defaults to `./out/operator/`).
That checkout supplies sample CRs and kustomize bases used by the root Makefile.

| Variable | Default | Purpose |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openstack-k8s-operators/install_yamls](https://github.com/openstack-k8s-operators/install_yamls) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

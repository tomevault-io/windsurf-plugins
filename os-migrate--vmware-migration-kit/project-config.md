---
trigger: always_on
description: > This document provides context for AI coding assistants working with this repository.
---

# VMware Migration Kit - AI Agent Context

> This document provides context for AI coding assistants working with this repository.

## Project Overview

**Name:** `os_migrate.vmware_migration_kit`  
**Version:** 2.2.0  
**License:** Apache-2.0  
**Repository:** https://github.com/os-migrate/vmware-migration-kit

This is an Ansible collection for migrating virtual machines from VMware (ESXi/vCenter) to OpenStack clouds. It uses a **hybrid architecture** combining:
- **Ansible roles & playbooks** for orchestration and workflow management
- **Go binaries** for high-performance migration operations (compiled to native executables)
- **Python wrappers** for Ansible module interface compatibility

## Architecture

### Directory Structure

```
vmware-migration-kit/
├── galaxy.yml                    # Ansible collection metadata (source of truth for version)
├── Makefile                      # Build automation (containers, tests, binaries)
├── go.mod / go.sum               # Go module dependencies
├── playbooks/                    # Main entry point playbooks
│   ├── migration.yml             # Primary NBDKit-based migration
│   ├── migration_v2v.yml         # Virt-v2v workflow
│   └── ...
├── roles/
│   ├── prelude/                  # Setup and validation
│   ├── conversion_host/          # OpenStack conversion host deployment
│   ├── export_metadata/          # VMware metadata extraction
│   ├── convert_metadata/         # Metadata transformation (VMware → OpenStack)
│   └── import_workloads/         # VM import and disk migration
├── plugins/
│   ├── modules/                  # Ansible modules (Python wrappers + Go binaries)
│   │   ├── *.py                  # Python wrapper files (DOCUMENTATION, thin interface)
│   │   ├── migrate               # Compiled Go binary (no extension)
│   │   ├── create_server         # Compiled Go binary
│   │   └── src/                  # Go source code (not shipped in collection)
│   │       ├── migrate/
│   │       │   └── migrate.go
│   │       ├── create_server/
│   │       │   └── create_server.go
│   │       └── ...
│   └── module_utils/             # Shared utilities
│       ├── vmware/               # govmomi vCenter/ESXi integration
│       ├── openstack/            # Gophercloud OpenStack integration
│       ├── nbdkit/               # NBD server management
│       ├── logger/               # Structured logging (logrus)
│       ├── ansible/              # Ansible module interface helpers
│       ├── connectivity/         # Network connectivity checks
│       └── utils.go              # Common utilities
├── tests/
│   ├── unit/                     # Go unit tests (*_test.go)
│   ├── integration/              # Ansible integration tests
│   └── sanity/                   # Ansible sanity ignore files
├── scripts/
│   └── build.sh                  # Containerized Go build script
└── aee/                          # Ansible Execution Environment config
```

### Component Relationships

```
┌─────────────────────────────────────────────────────────────────┐
│                     Ansible Controller                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ playbooks/   │  │ roles/       │  │ vars.yaml    │          │
│  └──────┬───────┘  └──────┬───────┘  └──────────────┘          │
│         │                 │                                     │
│         └────────┬────────┘                                     │
│                  ▼                                               │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              plugins/modules/*.py (wrappers)              │ │
│  │    (DOCUMENTATION + argument passing to Go binaries)      │ │
│  └─────────────────────────┬─────────────────────────────────┘ │
│                            ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │           plugins/modules/<binary> (Go executables)       │ │
│  │      migrate | create_server | flavor_info | ...          │ │
│  └─────────────────────────┬─────────────────────────────────┘ │
│                            │                                    │
└────────────────────────────┼────────────────────────────────────┘
                             ▼
┌────────────────────────────────────────────────────────────────┐
│                    Conversion Host (OpenStack VM)              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   nbdkit    │  │  virt-v2v   │  │  virtio-win │            │
│  │   server    │  │ (optional)  │  │  (Windows)  │            │
│  └──────┬──────┘  └─────────────┘  └─────────────┘            │
│         │                                                      │
│    ┌────┴────┐                                                 │
│    │ NBD/VDDK│────────────────────────────────────────────────┼──► VMware ESXi/vCenter
│    └─────────┘                                                 │      (port 902/TCP)
│         │                                                      │
│    ┌────┴────┐                                                 │
│    │ Cinder  │────────────────────────────────────────────────┼──► OpenStack APIs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [os-migrate/vmware-migration-kit](https://github.com/os-migrate/vmware-migration-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->

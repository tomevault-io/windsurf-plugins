---
trigger: always_on
description: Pantavisor is a container-based runtime for embedded Linux systems. It handles container orchestration, atomic OTA updates, remote management, and inter-container communication.
---

# Pantavisor

Pantavisor is a container-based runtime for embedded Linux systems. It handles container orchestration, atomic OTA updates, remote management, and inter-container communication.

## Key Components

| Component | Description |
|-----------|-------------|
| `xconnect/` | Service mesh daemon with plugins (unix, rest, dbus, drm, wayland) |
| `ctrl/` | REST API: /xconnect-graph, /daemons, /signal, /containers, /groups endpoints |
| `tools/pvcurl` | Lightweight curl wrapper using nc for HTTP-over-Unix-socket |
| `tools/pvcontrol` | CLI wrapper around pvcurl for pv-ctrl operations |
| `utils/tsh.c` | Daemon stdout/stderr capture via logserver |

## Documentation

### Reference (`docs/reference/`)

API and format specifications, versioned with each Pantavisor release. Always update when modifying the corresponding feature.

| Document | Location | Description |
|----------|----------|-------------|
| **Control Socket** | [docs/reference/pantavisor-commands.md](docs/reference/pantavisor-commands.md) | pv-ctrl HTTP endpoints (containers → PV) |
| **xconnect** | [docs/reference/pantavisor-xconnect.md](docs/reference/pantavisor-xconnect.md) | Service mesh manifests and mediation patterns |
| **xconnect Spec** | [xconnect/XCONNECT.md](xconnect/XCONNECT.md) | Technical specification and plugin architecture |
| **Configuration** | [docs/reference/pantavisor-configuration.md](docs/reference/pantavisor-configuration.md) | All configuration keys, defaults, and levels |
| **Log Sockets** | [docs/reference/logserver-sockets.md](docs/reference/logserver-sockets.md) | Logserver unix sockets reference |
| **Metadata** | [docs/reference/pantavisor-metadata.md](docs/reference/pantavisor-metadata.md) | User and device metadata reference |
| **State Format** | [docs/reference/pantavisor-state-format-v2.md](docs/reference/pantavisor-state-format-v2.md) | state.json format (v2) |

### Tools (`docs/tools/`)

On-device CLI tool docs, versioned with each Pantavisor release alongside `docs/reference/`.

| Document | Location | Description |
|----------|----------|-------------|
| **Tools** | [docs/tools/pantavisor-tools.md](docs/tools/pantavisor-tools.md) | pventer, pvcurl, pvcontrol, pvtx — on-device CLI tools |
| **pvcontrol** | [docs/tools/pvcontrol.md](docs/tools/pvcontrol.md) | Full `pvcontrol` CLI reference with worked examples |

### Technical Overview (`docs/overview/`)

Feature overview intended to be read top-to-bottom as a book, versioned with each Pantavisor release alongside `docs/reference/`. Synced to the docs site under the "Technical Overview" section.

| Document | Location | Description |
|----------|----------|-------------|
| **Architecture** | [docs/overview/pantavisor-architecture.md](docs/overview/pantavisor-architecture.md) | High-level architecture and state machine |
| **Revisions** | [docs/overview/revisions.md](docs/overview/revisions.md) | Revision concept and state JSON structure |
| **BSP** | [docs/overview/bsp.md](docs/overview/bsp.md) | Kernel, modules, firmware, bootloader |
| **Containers** | [docs/overview/containers.md](docs/overview/containers.md) | Container runtime, groups, roles, status |
| **Updates** | [docs/overview/updates.md](docs/overview/updates.md) | Update flow, states, transitions |
| **Storage** | [docs/overview/storage.md](docs/overview/storage.md) | On-disk layout, logs, metadata, integrity |
| **Disks** | [docs/overview/disks.md](docs/overview/disks.md) | Disk types, dual mode, dm-crypt, boot sequence |
| **Remote Control** | [docs/overview/remote-control.md](docs/overview/remote-control.md) | Pantacor Hub client and remote controllers |
| **Local Control** | [docs/overview/local-control.md](docs/overview/local-control.md) | pv-ctrl socket, Pantabox, pvcontrol |
| **Configuration Levels** | [docs/overview/pantavisor-configuration-levels.md](docs/overview/pantavisor-configuration-levels.md) | Configuration levels and precedence |
| **Init Mode** | [docs/overview/init-mode.md](docs/overview/init-mode.md) | Embedded, standalone, appengine modes |
| **Watchdog** | [docs/overview/watchdog.md](docs/overview/watchdog.md) | Watchdog configuration and modes |
| **Hooks** | [docs/overview/hooks.md](docs/overview/hooks.md) | System lifecycle hooks |
| **Inter-Container Communication** | [docs/overview/xconnect.md](docs/overview/xconnect.md) | xconnect service mesh overview |

## Docs Pipeline

The `docs/` folder is published on [docs.pantavisor.io/reference](https://docs.pantavisor.io/reference) by the [docs.pantavisor](https://github.com/pantavisor/docs.pantavisor) Docusaurus site, versioned per release rather than by folder:
1. Each meta-pantavisor release publishes a docs tarball, tracked in [`releases.json`](https://pantavisor-ci.s3.amazonaws.com/meta-pantavisor/releases.json) on S3. The tarball bundles this repo's `docs/` as `pantavisor/` and meta-pantavisor's `docs/` as a sibling `meta-pantavisor/` directory.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pantavisor/pantavisor](https://github.com/pantavisor/pantavisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

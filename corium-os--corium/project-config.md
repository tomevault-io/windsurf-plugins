---
trigger: always_on
description: Operating instructions for AI agents and human contributors working in this repository.
---

# AGENTS.md

Operating instructions for AI agents and human contributors working in this repository.

---

## 1. Language policy — non-negotiable

**Everything committed to this repository is written in English.** No exceptions.

This applies to:

- Source code: identifiers, function names, variable names, package names.
- Comments and docstrings.
- Log messages, error strings, CLI help text, and user-facing output.
- Documentation, README files, ADRs, and diagrams.
- Commit messages, branch names, pull request titles and bodies, issue titles and bodies.
- Test names and test fixtures.
- YAML/TOML keys, JSON Schema descriptions, and configuration examples.

Conversations with maintainers may happen in any language. The artefacts never do.

Rationale: Corium is infrastructure software intended for an international audience. A codebase with mixed-language identifiers is unreviewable and unmaintainable.

---

## 2. What Corium is

Corium is an immutable, container-native Linux distribution that boots directly into a
Kubernetes node. It combines three things:

1. **Fedora bootc** as the base OS (`quay.io/fedora/fedora-bootc`). The operating system
   *is* a container image: built with a `Containerfile`, pushed to a registry, versioned by
   digest, scanned with ordinary container tooling, and installed with `bootc-image-builder`.
2. **k0s** as the Kubernetes distribution, baked into the read-only `/usr` and upgraded by
   rolling a new OS image, not by mutating the running system.
3. **A cloud-init abstraction layer** that exposes a small, declarative, high-level `corium:`
   configuration surface for the common cases, while leaving the full raw cloud-init and
   k0s configuration reachable as an escape hatch.

The design goal, stated as a constraint: **a node should be describable in twenty lines of
YAML, and every one of those lines should be optional.**

### Non-goals

- Corium is not a fleet-management control plane. It provisions nodes; it does not manage them.
- Corium does not fork, patch, or vendor k0s. It configures and packages upstream k0s.
- Corium does not invent a new configuration language. It extends cloud-config.

---

## 3. Architectural decisions

These are settled. Changing one requires an ADR in `docs/adr/` and an explicit decision from
the maintainers — do not quietly work around them.

| # | Decision | Rationale |
|---|----------|-----------|
| 1 | Base image is **`quay.io/fedora/fedora-bootc`**, derived via `Containerfile` | OSTree/composefs atomic updates with cloud-init as the idiomatic first-boot surface. Fedora CoreOS was evaluated and rejected: it is Ignition-native, and layering cloud-init on it means neutralising Zincati and Afterburn to reach a place fedora-bootc already is |
| 2 | Kubernetes distribution is **k0s** | Single static binary, zero host dependencies, all state confined to `/var/lib/k0s`, control plane isolated from workloads by default |
| 3 | The k0s binary lives in **`/usr`** (read-only, image-owned) | Immutability is the product. Kubernetes upgrades ship as a new OS image |
| 4 | Kubernetes upgrades are **image-based**: new image → `bootc upgrade` → reboot → atomic rollback available | One upgrade mechanism, one version axis, free rollback |
| 5 | k0s **Autopilot is disabled** | Autopilot mutates the k0s binary in place, which contradicts decision 3 |
| 6 | Configuration is resolved from a **chain of sources**, cloud-init first among them | cloud-init covers every cloud and hypervisor, but bare metal, PXE and appliances have no datasource. `/etc/corium/config.yaml`, the kernel command line and an image default cover the rest |
| 7 | **No Ignition.** cloud-init is the only first-boot mechanism | Two provisioning systems on one node means two authorities over users, SSH keys and networking, and races between them |
| 8 | `corium-agent` and all tooling are written in **Go** | Same ecosystem as k0s and Kubernetes; static binaries drop cleanly into a read-only `/usr` |
| 9 | **Every abstraction has an escape hatch** | `corium:` covers the common path; raw `write_files`, `runcmd`, and a verbatim k0s config patch must always remain available |
| 10 | **Software RAID covers spare disks, not the root filesystem** | A root array is an install-time decision the `corium:` block is read too late to make, and the bootc path for one is broken upstream. See [ADR 3](docs/adr/0003-software-raid-scope.md) |
| 11 | **The management API is node-local, off by default, and a node it has not claimed is in no cluster** | One daemon per node answering for that node keeps the fleet-management non-goal intact. Off by default so no node in service grows a listening port by being upgraded. Holding the bootstrap until enrolment removes the state where a machine is both valuable and unclaimed, rather than defending it. See [ADR 4](docs/adr/0004-management-api.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Corium-OS/Corium](https://github.com/Corium-OS/Corium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

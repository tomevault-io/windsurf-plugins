---
trigger: always_on
description: This repository is a Fedora-first X11 desktop environment built around a
---

# AGENTS.md

## Purpose

This repository is a Fedora-first X11 desktop environment built around a
heavily patched fork of suckless dwm and a managed Quickshell shell. The
primary product is a complete desktop installed from Fedora Server Network
Install media. The existing Debian-, Arch-, Fedora-, and RHEL-family installer
remains a supported secondary path for the core desktop experience.

Read `SPEC.md` before making product, portability, installer, dependency, or
packaging changes. Treat `SPEC.md` as the source of truth for project scope and
acceptance criteria.

## Priorities

1. Preserve dwm stability and existing user workflows.
2. Keep the C window-manager core small, understandable, and dependency-light.
3. Build a cohesive Fedora desktop without moving desktop policy into the C
   event loop.
4. Preserve the core install and session contract across the three supported
   distribution families.
5. Keep installation and settings changes safe, repeatable, reversible, and
   explicit.
6. Prefer focused changes that can be reviewed and tested independently.

## Supported Platforms

The platform contract has two tiers:

- Primary desktop image: the current documented Fedora Server Network Install
  release, with separate standard and NVIDIA variants.
- Secondary existing-system install: these distribution families:

- Debian family: Debian, Ubuntu, Linux Mint, Pop!_OS, and compatible
  derivatives using `apt`.
- Arch family: Arch Linux, EndeavourOS, Manjaro, Arch Linux ARM, and compatible
  derivatives using `pacman`.
- RHEL family: RHEL, Rocky Linux, AlmaLinux, Fedora, and compatible
  derivatives using `dnf`.

Do not claim full desktop-environment parity on a secondary platform unless its
settings providers and runtime behavior were tested there. Core support still
requires dependency discovery, installation, X session startup, configuration
deployment, and the core runtime checks in `SPEC.md`; a successful C build is
not sufficient.

## Repository Map

- `dwm.c`, `drw.c`, `util.c`, `tomlparser.c`: window-manager sources.
- `config.def.h`: version-controlled default compile-time configuration.
- `config.h`: local build configuration. Do not overwrite user changes.
- `config.mk`: compiler, include, library, and installation settings.
- `Makefile`: build, install, uninstall, and release targets.
- `config/`: application configuration and default TOML runtime settings.
- `scripts/`: session startup, dependency checks, desktop helpers, and
  operational scripts.
- `install.sh`: supported existing-system installer for all distribution
  families.
- `dwm-fedora.ks`, `dwm-fedora-nvidia.ks`: Fedora image installation profiles.
- `dwm.desktop`: display-manager X session entry.
- `AGENTS.md`: durable engineering and agent-execution rules.
- `SPEC.md`: product scope, interfaces, and acceptance criteria.
- `ROADMAP.md`: ordered desktop-environment outcomes.
- `TASKS.md`: implementation work for the active roadmap phase only.
- `docs/`: user, contributor, and release documentation.

## Planning Workflow

- Use `SPEC.md` for durable product requirements and compatibility contracts.
- Use `ROADMAP.md` for ordered phase objectives and exit criteria.
- Use `TASKS.md` only for detailed work in the active phase. Replace its task
  set when a phase completes instead of accumulating historical checklists.
- Record completed user-visible behavior in `CHANGELOG.md` and releases.
- Do not mark a task or phase complete without its required validation or a
  precise statement of what could not be tested.
- Treat phase boundaries as review and rollback points. Do not begin the next
  phase in a change that was scoped only to complete the current one.

## Fedora Image Rules

- Base released images on the Fedora Server Network Install ISO documented in
  `SPEC.md` and `docs/RELEASING.md`, not on a Fedora Live image.
- Preserve separate standard and NVIDIA Kickstarts. Proprietary NVIDIA changes
  belong only to the explicitly selected NVIDIA image.
- Keep Kickstart package capabilities aligned with the shared dependency map.
- Run `make check-kickstart` for Kickstart or ISO-builder changes, then validate
  a real or virtual install before claiming the image boots or reaches a usable
  desktop.
- Record the Fedora release, source image checksum, architecture, firmware
  mode, image variant, and untested hardware in release evidence.
- The dedicated Fedora images currently set SELinux disabled by explicit
  product policy. Existing-system installs must not change host SELinux state.
  Any change to this policy requires a specification and migration update.

## Desktop Settings Rules

- Keep the Settings frontend and all QML unprivileged.
- Read state through stable service APIs, D-Bus, signals, subscriptions, or
  bounded helpers. Do not parse human-oriented output when a machine interface
  exists.
- Separate read-only state, user-session changes, privileged system changes,
  delegated tools, and unsupported capabilities.
- Privileged helpers must be installed root-owned and non-writable, expose only
  allowlisted operations, validate every argument, and require explicit user
  intent through polkit or an equally narrow authorization path.
- Repository or user-writable helper copies must never be elevated.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChrisTitusTech/dwm-titus](https://github.com/ChrisTitusTech/dwm-titus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

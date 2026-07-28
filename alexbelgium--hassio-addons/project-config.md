---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Home Assistant add-on repository containing 120+ Docker-based add-ons for the Home Assistant Supervisor. Each add-on is a self-contained directory with a Dockerfile, config schema, and S6-overlay init scripts. The repository uses GitHub Actions for CI/CD, linting, and automated upstream version tracking.

## Add-On Directory Structure

Most add-ons follow this common layout, though exceptions exist (e.g. some archived add-ons use `config.json` instead of `config.yaml`, some add-ons have `build.yaml` instead of `build.json` or no build file at all, and not every add-on includes a `rootfs/` tree):

```
addon_name/
├── config.yaml          # HA add-on metadata, schema, ports, maps
├── build.json           # Base Docker images per architecture (may be build.yaml, or absent)
├── Dockerfile           # Multi-stage build (always uses shared .templates/ scripts)
├── updater.json         # Upstream release tracking; required to enable automatic updates
├── CHANGELOG.md         # Required; must be updated on every PR
└── rootfs/              # Optional; absent in some add-ons
    └── etc/
        ├── cont-init.d/ # S6-overlay init scripts (numbered, run in order)
        └── services.d/  # S6-overlay supervised services (some add-ons use
                         # s6-overlay v3 layout at etc/s6-overlay/s6-rc.d/ instead)
```

## Dockerfile Convention

Most Dockerfiles follow this 6-section pattern (some add-ons deviate slightly, e.g. using a pinned upstream image directly instead of `ARG BUILD_FROM`):

1. **Build Image** – `ARG BUILD_FROM` + `FROM ${BUILD_FROM}`
2. **Modify Image** – S6 env vars, LSIO modifications via `ha_lsio.sh`
3. **Install Apps** – Copy `rootfs/`, download modules, install packages
4. **Entrypoint** – Set `S6_STAGE2_HOOK=/ha_entrypoint.sh`
5. **Labels** – Standard OCI + HA labels from build args
6. **Healthcheck** – curl-based check suppressed from nginx/apache logs

Shared build-time scripts are pulled from `.templates/` at build time:
- `ha_automodules.sh` – Downloads module scripts listed in `ARG MODULES=`
- `ha_autoapps.sh` – Installs packages listed in `ENV PACKAGES=`
- `ha_entrypoint.sh` – S6 stage-2 hook; launches the cont-init stack at container start
- `ha_lsio.sh` – Patches LinuxServer.io base images for HA compatibility
- `bashio-standalone.sh` – Bashio library for scripts outside Supervisor context

The `ARG MODULES=` line lists template scripts to download at build time (e.g., `00-banner.sh 01-custom_script.sh 00-smb_mounts.sh`). Commonly-used modules in `.templates/` (not exhaustive):
- `00-banner.sh` – Print the add-on startup banner
- `00-global_var.sh` – Initialize global env vars from HA options
- `00-local_mounts.sh` – Mount local disks (localdisks option)
- `00-smb_mounts.sh` – SMB/CIFS network mount support
- `00-deprecated.sh` – Print a deprecation warning for add-ons superseded by official ones
- `01-config_yaml.sh` – Map HA options → app's `config.yaml`
- `01-custom_script.sh` – Run user-provided custom scripts
- `19-json_repair.sh` – Validate/repair JSON config files
- `90-disable_ingress.sh` – Allow disabling HA ingress
- `90-dns_set.sh` – Configure custom DNS
- `91-silent.sh` – Reduce log verbosity
- `91-universal_graphic_drivers.sh` – GPU driver detection
- `99-custom_script.sh` – Run a user `script.sh` from the add-on config dir at startup

Other helper scripts in `.templates/` used at build/run time: `ha_automatic_packages.sh` (resolve package names across distros), `ha_entrypoint_modif.sh`, `00-aaa_dockerfile_backup.sh`, plus `config.template`/`script.template`/`show_text_color` (templates/assets copied into add-ons).

## config.yaml Schema

Key fields in every add-on's `config.yaml`:

```yaml
arch: [aarch64, amd64]
image: ghcr.io/alexbelgium/{slug}-{arch}
version: "X.Y.Z"          # upstream version (format varies; see Versioning section)
ingress: true/false
ingress_port: 8000
map:
  - addon_config:rw        # /addon_configs/<hostname>/
  - share:rw
  - media:rw
  - ssl
schema:
  env_vars:                # Allows arbitrary env var passthrough
    - name: match(^[A-Za-z0-9_]+$)
      value: str?
  PUID: int
  PGID: int
  TZ: str?
  networkdisks: str?       # SMB mounts
  localdisks: str?         # Local disk mounts
```

The `env_vars` schema key enables the env-var passthrough mechanism. At runtime the `00-global_var.sh` cont-init module reads `/data/options.json` and exports each key as an environment variable (writing to `/.env` and `/etc/environment`). `ha_entrypoint.sh` is the S6 stage-2 hook that launches the cont-init stack but does not itself perform the JSON-to-env conversion.

## Versioning

Add-on versions in `config.yaml` closely follow the upstream release tag and do not conform to a single fixed format. Common patterns include:

- `X.Y.Z` – plain upstream semver (e.g. `0.137.0`)
- `X.Y.Z.N` – upstream version with a local patch counter (e.g. `0.6.26.2`)
- LSIO-style tags (e.g. `1.43.1.10611-1e34174b1-ls301`)
- Date-based versions (e.g. `2026.02.28`)
- Nightly builds (e.g. `nightly-20260321-397`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexbelgium/hassio-addons](https://github.com/alexbelgium/hassio-addons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

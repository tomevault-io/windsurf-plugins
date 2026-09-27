---
trigger: always_on
description: This repository publishes VoHive v1.5.5 one-click installers for Linux. It does not contain the VoHive application source. The application binaries are opaque upstream artifacts packaged for `amd64` and `arm64`.
---

# Repository Instructions

## Purpose and scope

This repository publishes VoHive v1.5.5 one-click installers for Linux. It does not contain the VoHive application source. The application binaries are opaque upstream artifacts packaged for `amd64` and `arm64`.

These instructions apply to the entire repository.

## Supported behavior

- Target only Linux systems using systemd.
- Support both `amd64`/`x86_64` and `arm64`/`aarch64`.
- Keep the online path as one command through `install.sh`.
- Keep each offline `.run` file self-contained and network-free when executed directly.
- Install under `/opt/vohive` and register `/etc/systemd/system/vohive.service`.
- Preserve an existing `/opt/vohive/config/config.yaml` during reinstall or upgrade.
- Back up an existing binary to `/opt/vohive/bin/vohive.bak` before replacement.
- Install `/opt/vohive/uninstall.sh`. Interactive execution must ask whether to remove config, data, and logs, defaulting to preservation. Non-interactive execution and `--keep-data` must preserve them; `--purge` may remove them without prompting.
- Never execute the README's modem VID/PID-changing AT commands from an installer. They modify hardware-persistent state and must remain an explicit manual operation.

## Repository map

- `README.md`: user-facing VID/PID, online installation, offline installation, and maintenance instructions.
- `install.sh`: POSIX-shell online bootstrap. It detects the CPU architecture, downloads the matching `.run` file, verifies the top-level SHA-256, installs dependencies, and executes the installer. GitHub Raw failures fall back to `ghfast.top`.
- `uninstall.sh`: POSIX-shell uninstaller. It interactively asks whether to clear state, supports explicit `--keep-data` and `--purge`, and defaults to preserving state when non-interactive.
- `packaging/install-offline.sh`: Bash systemd installer embedded in both offline packages.
- `packaging/self-extract.sh`: POSIX-shell header prepended to each compressed payload.
- `packaging/config.default.yaml`: default configuration installed only when no configuration exists.
- `packaging/vohive.service`: systemd unit installed on the target.
- `packaging/mcc-mnc-table.json`: MCC/MNC-to-country-and-operator lookup data installed at `/opt/vohive/data/mcc-mnc-table.json`.
- `scripts/build-offline-installers.sh`: builds both self-extracting `.run` files from explicitly supplied binaries and rewrites `checksums.txt`.
- `tests/test-installers.sh`: validates shell syntax, top-level checksums, payload extraction, internal checksums, and ELF architectures.
- `vohive-offline-v1.5.5-linux-{amd64,arm64}.run`: generated release artifacts; do not edit by hand.
- `checksums.txt`: generated SHA-256 values for the two `.run` artifacts.

## Shell and portability rules

- `install.sh`, `uninstall.sh`, and `packaging/self-extract.sh` must remain POSIX `sh` compatible.
- `packaging/install-offline.sh`, the builder, and tests may use Bash.
- Do not add dependencies when standard shell tools are sufficient.
- Do not hardcode developer-specific absolute paths. The build script must continue to require both binary paths as arguments.
- Quote paths and variables. Keep `set -eu` or `set -euo pipefail` enabled as appropriate.
- The online bootstrap's temporary-directory signal traps must exit after `INT` or `TERM`; never clean the directory and then continue execution.
- Online mode sets `VOHIVE_INSTALL_DEPS=1` and installs `socat`, `usbutils`, and `pciutils` with `apt-get`, `dnf`, or `yum`.
- Direct execution of an offline `.run` file must not contact package repositories or any other network endpoint. It requires Bash, `awk`, `tail`, `tar`, `sha256sum`, and systemd; `socat`, `lsusb`, and `lspci` are optional tools that it only reports as missing.

## Generated artifacts and release invariants

The version appears in filenames, `install.sh`, the builder, and documentation. A version change must update all of them together.

Never modify a `.run` file or `checksums.txt` manually. Rebuild both architectures whenever any of these inputs changes:

- either VoHive binary;
- any file under `packaging/`;
- `uninstall.sh`;
- the version or generated filename format.

Do not rebuild or modify the `.run` files and `checksums.txt` for changes limited to `README.md`, `AGENTS.md`, the online `install.sh`, or tests. Those files are not part of the offline payload. Binary artifacts should change only when an embedded payload input or its packaging format actually changes.

Rebuild with explicit inputs:

```bash
./scripts/build-offline-installers.sh \
  /path/to/vohive_v1.5.5_linux_amd64 \
  /path/to/vohive_v1.5.5_linux_arm64
```

The builder must reject binaries whose ELF architecture does not match the supplied slot. It must regenerate both root checksums and the checksums embedded in each payload.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sdrpsps/vohive-oneclick](https://github.com/sdrpsps/vohive-oneclick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

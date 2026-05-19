---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Role

**Production** — stable kernel, tested packages, released to users. Paired with `kiro-calamares-config`.

| Repo | Role | Calamares config |
|---|---|---|
| `kiro-iso` | **Production** — stable kernel, tested packages, released to users | `kiro-calamares-config` |
| `kiro-iso-next` | **Beta/Testing** — experimental features, kernel changes, new packages under evaluation | `kiro-calamares-config-next` |

## Project

Custom Arch Linux ISO builder based on ArchISO. Produces a live/installable ISO with XFCE4 + ohmychadwm desktop, pre-configured packages, and systemd optimizations.

## Build Workflow

Always run these in order from `build-scripts/`:

```bash
# 1. Bump version across all version files (generates vYY.MM.DD.01)
bash change-version.sh

# 2. Build the ISO (run as normal user — script calls sudo internally)
cd build-scripts && bash build-the-iso.sh
```

- Build output lands in `~/kiro-Out/`
- Build working directory is `~/kiro-build/` (deleted/recreated each run)
- Checksums (sha1, sha256, md5) and a pkglist are auto-generated alongside the ISO

**Do not run `build-the-iso.sh` as root.**

## Version Files

`change-version.sh` updates the version string (`vYY.MM.DD.01`) in exactly these three places — keep them in sync:

| File | Field |
|---|---|
| `archiso/airootfs/etc/dev-rel` | `ISO_RELEASE=` |
| `archiso/profiledef.sh` | `iso_label=` and `iso_version=` |
| `build-scripts/build-the-iso.sh` | `kiroVersion=` |

To bump the `.01` suffix for same-day rebuilds, edit `extra="01"` in `change-version.sh`.

## Nvidia Driver Selection

In `build-scripts/build-the-iso.sh`, set the `nvidia_driver` variable in the **config block at the top of the file** before building:

- `open` — nvidia-open-dkms (default, modern GPUs)
- `580xx` — nvidia-580xx-dkms (legacy)
- `390xx` — nvidia-390xx-dkms (legacy)

The script manipulates `packages.x86_64` in the build folder to inject the chosen driver set.

## Architecture

The build pipeline:
1. `build-the-iso.sh` copies `archiso/` into `~/kiro-build/archiso/`
2. Fetches latest `.bashrc` from `erikdubois/edu-shells` into `airootfs/etc/skel/`
3. Pre-populates the pacman GPG keyring (archlinux + chaotic) in the build tree
4. Injects the correct NVIDIA packages into the package list
5. Calls `mkarchiso` to squash and produce the ISO

`archiso/airootfs/` is the overlay applied on top of the base Arch system — files here end up at `/` on the live ISO. Key subdirectories:
- `etc/` — system config (pacman, NetworkManager, locale, hostname, polkit, modprobe)
- `root/` — root user's home on the live system
- `usr/` — additional binaries/configs

## Package Repositories

Defined in `archiso/pacman.conf` (used during ISO build) and `build-scripts/pacman.conf`:

- `[core]` / `[extra]` / `[multilib]` — standard Arch mirrors
- `[kiro_repo]` — `https://kirodubes.github.io/$repo/$arch` (SigLevel Never)
- `[nemesis_repo]` — `https://erikdubois.github.io/$repo/$arch` (SigLevel Never)
- `[chaotic-aur]` — requires `chaotic-keyring` + `chaotic-mirrorlist` on the build host
- `[personal_repo]` — optional local repo, commented out by default (see in-file comment for path)

## Key Files

- `archiso/airootfs/etc/dev-rel` — ISO version string (`ISO_RELEASE=`, `ISO_CODENAME=`, `ISO_BUILD=`)
- `archiso/packages.x86_64` — full package list (one package per line, comments with `#`)
- `archiso/profiledef.sh` — ArchISO profile: name, label, version, bootmodes, compression
- `archiso/pacman.conf` — pacman config used inside the ISO build
- `archiso/efiboot/loader/entries/` — UEFI boot entries (kernel + initrd paths; must match kernel in packages.x86_64)
- `build-scripts/build-the-iso.sh` — full build pipeline
- `build-scripts/get-pacman-repos-keys-and-mirrors.sh` — installs chaotic-keyring/mirrorlist if missing
- `change-version.sh` — version bump script
- `up.sh` — git pull → `git add --all` + commit `"update"` + push; quick-push only, not for structured commits
- `audit.sh` — installed system health checker; run on a freshly installed Kiro VM to verify all Calamares modules ran correctly

## isoLabel Must Match profiledef.sh

`isoLabel` in `build-the-iso.sh` is constructed as `kiro-${kiroVersion}-x86_64.iso`. It must start with `iso_name` from `profiledef.sh` (`kiro`) — mismatch causes the checksum phase to fail with "No such file or directory".

## Security Baseline

A full Arch vs Kiro security comparison was run 2026-05-19 — results in **`ARCH-VS-KIRO-SECURITY.md`**. All action items resolved:

- `archiso/airootfs/etc/ssh/sshd_config.d/10-archiso.conf` — **kept intentionally**. The `sshd_config.d/` directory must have at least one file or archiso won't create it on the live ISO, causing errors. The file enables root SSH for the live session; `kiro_final` removes it from the installed system. Confirmed absent post-install.
- `archiso/airootfs/etc/tmpfiles.d/cups-permissions.conf` — **added**. Enforces `600 root:cups` on CUPS config files at boot via `systemd-tmpfiles`.
- No firewall — **by design**. `iptables` is installed but intentionally has no rules.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kirodubes/kiro-iso](https://github.com/kirodubes/kiro-iso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->

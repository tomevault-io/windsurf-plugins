---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`sdbootutil` is a bootctl wrapper for BLS (Boot Loader Specification) boot loaders (systemd-boot and grub2-bls) designed for btrfs-based, snapper-managed systems. It manages the full lifecycle of bootloader installations with Full Disk Encryption (FDE) support based on systemd.

**Key Capabilities:**
- Installs and updates systemd-boot with shim for secure boot
- Manages kernel entries in ESP (EFI System Partition) with snapshot awareness
- Handles btrfs snapshots via snapper integration
- Implements full disk encryption enrollment (TPM2, FIDO2, password)
- Uses checksums for kernel/initrd deduplication across snapshots
- Manages PCR (Platform Configuration Register) predictions for TPM2

## Architecture

### Core Components

1. **`sdbootutil` (main bash script)** - 4400+ lines
   - Primary interface for all bootloader operations
   - Handles kernel/entry management, snapshot integration, FDE enrollment
   - Entry point for snapper hooks and RPM triggers

2. **`uhmac/` (Rust utility)** - HMAC calculation utility
   - Used for cryptographic operations in FDE scenarios
   - Built with cargo, linked as `%{_libexecdir}/sdbootutil/uhmac`

3. **Integration Scripts:**
   - `10-sdbootutil.snapper` - Snapper plugin for snapshot lifecycle hooks
   - `kernelhooks.lua` - RPM file trigger for kernel package install/remove
   - `10-sdbootutil.tukit` - Tukit plugin for transactional systems
   - `50-sdbootutil.install` - kernel-install plugin script

4. **FDE/TPM Components:**
   - `measure-pcr-generator.sh` - Generates PCR 15 predictions
   - `measure-pcr-validator.sh` - Validates PCR measurements at boot
   - `sdbootutil-enroll` - Enrollment service wrapper
   - `jeos-firstboot-enroll` - JEOS integration for first-boot enrollment

### How Snapshots Work

Unlike standard systemd-boot which assumes one OS instance per kernel version, sdbootutil extends this for btrfs snapshots where multiple subvolumes share kernels:

- **Entry naming:** `{machine-id}-{version}-{snapshot}.conf` (e.g., `2ceda9f-6.2.1-1-default-15.conf`)
- **Kernel/initrd deduplication:** Uses checksums instead of snapshot numbers in filenames
  - Example: `linux-b021b508eb42b2afd06de8f0242b9727aa7dc494`
  - Allows multiple snapshots to share same kernel/initrd files
- **Reference counting:** Uses `bootctl unlink` and `bootctl cleanup` to safely remove entries
- **Initrd reuse:** Intelligently reuses initrds from parent snapshots when possible

### System Integration Points

**Snapper Hooks** (`10-sdbootutil.snapper`):
- `create-snapshot-post`: Updates bootloader, adds entries (Tumbleweed) or defers to set-default (transactional)
- `delete-snapshot-pre`: Removes entries for deleted snapshot
- `set-default-snapshot-post`: Sets bootloader default, adds entries (transactional systems only)

**RPM Triggers** (`kernelhooks.lua`):
- Monitors `/usr/lib/modules/{version}/vmlinuz` installations
- Filters out legacy `/boot/vmlinuz-*` locations
- Calls `sdbootutil add-kernel` / `remove-kernel` automatically
- Note: File triggers can be unreliable with zypper

**Transactional vs Non-Transactional:**
- **Transactional (MicroOS):** Kernel entries added in `set-default-snapshot-post` after transaction completes
- **Non-Transactional (Tumbleweed):** Kernel entries added immediately in `create-snapshot-post`

## Common Commands

### Build

```bash
# Build uhmac utility (Rust)
cd uhmac
cargo build --release

# For RPM build, see sdbootutil.spec (%build and %install sections)
```

### Testing sdbootutil

```bash
# Enable trace mode (outputs to /var/log/sdbootutil.log)
sudo ./sdbootutil --start-trace-code <command>
sudo ./sdbootutil --stop-trace-code

# Verbose output
sudo ./sdbootutil -v <command>

# Check bootloader status
sudo ./sdbootutil is-installed
sudo ./sdbootutil bootloader

# List entries/kernels for snapshot
sudo ./sdbootutil list-entries [snapshot]
sudo ./sdbootutil list-kernels [snapshot]
sudo ./sdbootutil list-snapshots

# Test kernel entry management
sudo ./sdbootutil add-kernel <version> [snapshot]
sudo ./sdbootutil remove-kernel <version> [snapshot]
sudo ./sdbootutil cleanup [snapshot]
```

### Full Disk Encryption Operations

```bash
# Enroll TPM2 with PIN
sudo ./sdbootutil enroll --method tpm2+pin

# Enroll FIDO2 key
sudo ./sdbootutil enroll --method fido2

# Update PCR predictions after kernel/bootloader changes
sudo ./sdbootutil update-predictions

# List tracked encrypted devices
sudo ./sdbootutil list-devices
```

### Bootloader Management

```bash
# Install bootloader with shim (secure boot)
sudo ./sdbootutil install --secure-boot

# Check if bootloader needs update
sudo ./sdbootutil needs-update

# Update bootloader (only if newer version available)
sudo ./sdbootutil update

# Force update bootloader to match system version
sudo ./sdbootutil update --sync
```

## Configuration

### Config File Hierarchy
1. `/etc/sdbootutil.conf` - User configuration (loaded if exists)
2. Auto-generated from defaults if bootloader installed but no config exists
3. CLI arguments override config file values

**Key Config Variables:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openSUSE/sdbootutil](https://github.com/openSUSE/sdbootutil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

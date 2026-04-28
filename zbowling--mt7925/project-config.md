---
trigger: always_on
description: This document helps AI agents effectively work with this repository. It provides critical requirements, workflow guidelines, and a documentation index for finding relevant information.
---

# Instructions for AI Agents

This document helps AI agents effectively work with this repository. It provides critical requirements, workflow guidelines, and a documentation index for finding relevant information.

## Documentation Index (Progressive Disclosure)

The `docs/` folder contains comprehensive documentation. Use this index to find the right file for your task.

### Quick Reference: What Do You Need?

| If you need to... | Read this file |
|-------------------|----------------|
| Understand the overall driver architecture | [ARCHITECTURE.md](docs/ARCHITECTURE.md) |
| Find where a specific function is defined | [FILE_REFERENCE.md](docs/FILE_REFERENCE.md) |
| Trace how a feature works end-to-end | [NAVIGATION_GUIDE.md](docs/NAVIGATION_GUIDE.md) |
| Debug a crash or hang | [DEBUGGING.md](docs/DEBUGGING.md) |
| Use debugfs entries (fw_debug, chip_reset, etc.) | [DEBUGFS.md](docs/DEBUGFS.md) |
| Use sysfs entries (temperature, rfkill, etc.) | [SYSFS.md](docs/SYSFS.md) |
| Understand MCU command timeouts | [MCU_PROTOCOL.md](docs/MCU_PROTOCOL.md) |
| Fix or understand MLO (Multi-Link) issues | [MLO.md](docs/MLO.md) |
| Understand or fix locking/deadlock issues | [LOCKING.md](docs/LOCKING.md) |
| Port patches between kernel versions | [PATCH_DIFFERENCES.md](docs/PATCH_DIFFERENCES.md) |
| Understand chipset differences | [CHIPSETS.md](docs/CHIPSETS.md) |

---

### Documentation by Category

#### Architecture & Code Understanding

| File | Contents | When to Read |
|------|----------|--------------|
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | Module layering (core → connac → mt792x → mt7925), design patterns, component relationships | Understanding how the driver is structured |
| [DATA_STRUCTURES.md](docs/DATA_STRUCTURES.md) | Core structs (`mt76_dev`, `mt792x_dev`, `mt76_wcid`, `mt792x_vif`), field meanings, relationships | Working with driver data structures |
| [FILE_REFERENCE.md](docs/FILE_REFERENCE.md) | Every source file organized by layer, what each file contains, line counts | Finding where specific code lives |
| [ENTRY_POINTS.md](docs/ENTRY_POINTS.md) | PCI probe, module init, mac80211 registration, initialization sequence | Understanding driver startup |
| [CONTROL_FLOW.md](docs/CONTROL_FLOW.md) | TX/RX packet paths, interrupt handling, work queue flows | Tracing runtime behavior |
| [NAVIGATION_GUIDE.md](docs/NAVIGATION_GUIDE.md) | How to trace features, grep patterns, common code paths | Finding related code quickly |

#### Debugging & Diagnostics

| File | Contents | When to Read |
|------|----------|--------------|
| [DEBUGGING.md](docs/DEBUGGING.md) | Common issues (NULL derefs, deadlocks, MCU timeouts), diagnostic commands, stress testing | Debugging any driver issue |
| [DEBUGFS.md](docs/DEBUGFS.md) | All debugfs entries: `fw_debug`, `chip_reset`, `regval`, queues, trace events, MCU command IDs | Using `/sys/kernel/debug/ieee80211/phy*/mt76/` |
| [SYSFS.md](docs/SYSFS.md) | Hwmon (temperature), rfkill, network stats, PCI device control, power management | Using `/sys/class/` interfaces |
| [lock-audit.md](docs/lock-audit.md) | Which functions acquire mutex, which expect it held, call patterns | Analyzing lock-related bugs |

#### Protocols & Features

| File | Contents | When to Read |
|------|----------|--------------|
| [MCU_PROTOCOL.md](docs/MCU_PROTOCOL.md) | MCU command format, TLV encoding, command IDs, response handling, timeout behavior | MCU communication issues |
| [MLO.md](docs/MLO.md) | Multi-Link Operation architecture, link structures, common NULL pointer bugs, link iteration | WiFi 7 / MLO issues |
| [KERNEL_INTERACTIONS.md](docs/KERNEL_INTERACTIONS.md) | mac80211 integration, PCI subsystem, DMA, power management, cfg80211 | Kernel API usage |
| [LOCKING.md](docs/LOCKING.md) | Mutex patterns, `mt792x_mutex_acquire`, work queue interactions, deadlock scenarios | Concurrency bugs |

#### Chipset & Version Specifics

| File | Contents | When to Read |
|------|----------|--------------|
| [CHIPSETS.md](docs/CHIPSETS.md) | All MediaTek WiFi chipsets (7603→7925), specs, release dates, driver mapping | Understanding hardware differences |
| [PATCH_DIFFERENCES.md](docs/PATCH_DIFFERENCES.md) | Function renames between kernel versions, API changes, porting guidance | Porting patches to new kernels |

#### Known Issues & Workarounds

| File | Contents | When to Read |
|------|----------|--------------|
| [6ghz-mlo-workaround.md](docs/6ghz-mlo-workaround.md) | 6GHz band not associating in MLO, ACPI/regulatory workaround | 6GHz not working with MLO |

---

## Critical Requirements

### Git Workflow (IMPORTANT)

**DO NOT push directly to `main` branch.** The repository has branch protection rules that block direct pushes.

1. **Always use feature branches**
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/your-bug-fix
   # or
   git checkout -b docs/your-documentation-change
   ```

2. **Push to feature branch and create a PR**
   ```bash
   git push -u origin feature/your-feature-name
   gh pr create --fill
   ```

3. **Branch naming conventions**
   - `feature/` - New features or enhancements
   - `fix/` - Bug fixes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zbowling/mt7925](https://github.com/zbowling/mt7925) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

---
trigger: always_on
description: **Compile**: `make -C drivers/aic8800` (auto-detects clang, uses LLVM if available)
---

# AIC8800 Linux Driver - Agent Guidelines

## Build System

**Compile**: `make -C drivers/aic8800` (auto-detects clang, uses LLVM if available)  
**Force GCC**: `make LLVM=0 -C drivers/aic8800` (only works if kernel was built with GCC)  
**Force clang**: `make LLVM=1 -C drivers/aic8800`  
**Clean**: `make -C drivers/aic8800 clean`  
**Install/Uninstall**: `make -C drivers/aic8800 {install,install_firmware,install_rules,install_modules,uninstall,uninstall_firmware,uninstall_rules,uninstall_modules}`

**IMPORTANT**: Out-of-tree modules must use the same compiler as the kernel. If the kernel was built with clang (e.g. CachyOS, Arch), GCC builds will fail with unrecognized flag errors (`-mstack-alignment`, `-mllvm`, `-fsplit-lto-unit`). The auto-detection handles this correctly.

**LLVM auto-detection** (in `drivers/aic8800/Makefile`):
- `LLVM` undefined + clang available → uses clang
- `LLVM=1` → forces clang
- `LLVM=0` → forces GCC (stripped from MAKEOVERRIDES to prevent leaking to kbuild via MAKEFLAGS, since kbuild's `ifdef LLVM` treats any non-empty value as true)

**Platform configs**: `CONFIG_PLATFORM_ROCKCHIP` (arm64), `CONFIG_PLATFORM_ALLWINNER` (arm64), `CONFIG_PLATFORM_AMLOGIC` (arm), `CONFIG_PLATFORM_HI` (arm), `CONFIG_PLATFORM_UBUNTU` (default)

## Code Style

- Linux kernel style (8-space tabs, 80-char lines)
- `.h` files: declarations; `.c` files: implementations
- License: GPL-2.0 (`MODULE_LICENSE("GPL")` in both modules, `LICENSE` file at repo root)
- Original copyright: `Copyright (C) RivieraWaves 2012-2019` and `Copyright (C) AICSemi 2018-2020`
- Kernel-doc: `/** ... */`
- Names: `rwnx_*` (core), `aicwf_*` (wifi)
- Include order: `<linux/.>`, `<net/.>`, `"rwnx_*.h"`, `"aicwf_*.h"`, `"reg_*.h"`

## Naming

- Types: `rwnx_*_t` (e.g., `rwnx_hw`, `rwnx_sta`)  
- Functions: `rwnx_*`, `aicwf_*`  
- Constants: `RWNX_*`, `AICWF_*`  
- Globals: `aic_fw_path`, `country_code`

## Error Handling

- Return negative errno (`-ENOMEM`, `-EINVAL`, `-1`)
- `RWNX_DBG(LOGDEBUG/LOGINFO/LOGERROR, ...)`  
- `printk(KERN_CRIT ...)` for critical errors  
- Validate firmware file size > 0

## Preprocessor

- `#ifdef CONFIG_*` for Kconfig  
- `#if LINUX_VERSION_CODE >= KERNEL_VERSION(...)` for version checks  
- Headers: `#ifndef _NAME_H_` / `#define _NAME_H_` / `#endif`

## Types

```c
u8, u16, u32, u64, s8, s16, s32, s64  // <linux/types.h>
bool                                  // Linux kernel
```

## Memory/Thread Safety

- `kzalloc()` / `kfree()`  
- `skb_queue_len()`, `skb_queue_walk()`  
- `spin_lock_bh()` / `spin_unlock_bh()`  
- `tasklet_schedule()`, `tx_lock` spinlock

## Module Config

Key options: `CONFIG_AIC8800_WLAN_SUPPORT=m`, `CONFIG_USB_SUPPORT=y`, `CONFIG_SDIO_SUPPORT=n`, `CONFIG_RWNX_FULLMAC=y`, `CONFIG_RWNX_SDM=n`, `CONFIG_RWNX_TL4=n`, `CONFIG_DEBUG_FS=n`, `CONFIG_RWNX_DBG=n` (production-safe default)

## Kconfig Location

- `drivers/aic8800/Kconfig`  
- `drivers/aic8800/aic8800_fdrv/Kconfig`  
- `drivers/aic8800/aic_load_fw/Kconfig`

## File Organization

```
drivers/aic8800/
├── aic8800_fdrv/   # WiFi driver (rwnx_*.c, aicwf_*.c)
├── aic_load_fw/    # Firmware (aicbluetooth_*.c, aicwf_*.c)
└── Makefile
```

Key files: `rwnx_main.c`, `rwnx_tx.c/rwnx_rx.c`, `rwnx_msg_tx.c/rwnx_msg_rx.c`, `rwnx_cmds.c`, `rwnx_cfgfile.c`, `aicwf_compat_*.c`

## Kernel Compatibility

- Tested: CachyOS kernel 6.19.12-1-cachyos (clang 22.1.x), Arch Linux kernel 6.17.1-arch1-1  
- Compatibility macros in `rwnx_compat.h`/`rwnx_defs.h`  
  `HIGH_KERNEL_VERSION=KERNEL_VERSION(6,0,0)`  
  `HIGH_KERNEL_VERSION2=KERNEL_VERSION(6,1,0)`  
  `HIGH_KERNEL_VERSION3=KERNEL_VERSION(6,3,0)`  
  `HIGH_KERNEL_VERSION4=KERNEL_VERSION(6,3,0)`  
  On Android: `HIGH_KERNEL_VERSION=KERNEL_VERSION(5,15,41)`

## Debugging

### Debug Macros

- `AICWFDBG(level, fmt, ...)` - Conditional debug based on `aicwf_dbg_level & level`
- `RWNX_DBG(fmt, ...)` - Equivalent to `AICWFDBG(LOGTRACE, fmt, ...)`
- `printk(KERN_CRIT ...)` - **Only for critical errors** (always visible)

### Log Levels

```c
#define LOGERROR    0x0001  // Always visible in production
#define LOGINFO     0x0002  // Hidden by default
#define LOGTRACE    0x0004  // Hidden by default
#define LOGDEBUG    0x0008  // Hidden by default
#define LOGDATA     0x0010  // Hidden by default
```

### Control Debug Level

```bash
# View current level
cat /sys/module/aic_load_fw/parameters/aicwf_dbg_level
cat /sys/module/aic8800_fdrv/parameters/aicwf_dbg_level

# Enable verbose (ERROR + INFO + TRACE + DEBUG = 15)
echo 15 > /sys/module/aic_load_fw/parameters/aicwf_dbg_level
echo 15 > /sys/module/aic8800_fdrv/parameters/aicwf_dbg_level

# Enable only errors (1) - production default
echo 1 > /sys/module/aic_load_fw/parameters/aicwf_dbg_level
echo 1 > /sys/module/aic8800_fdrv/parameters/aicwf_dbg_level
```

### Default Debug Levels (Production-Safe)

- `aic_load_fw`: `LOGERROR` (0x1) - only errors
- `aic8800_fdrv`: `LOGERROR` (0x1) - only errors

### Deprecated

- `printk()` - **deprecated**, use `AICWFDBG()` or `RWNX_DBG()`  
- Enable debug build: `CONFIG_RWNX_DBG=y` (default: only errors visible)

## Common Commands

```bash
modprobe cfg80211 aic_load_fw aic8800_fdrv
lsmod | grep aic
dmesg | tail -20
```

## Platform Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ronnyf/AIC8800-Linux-Driver](https://github.com/ronnyf/AIC8800-Linux-Driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->

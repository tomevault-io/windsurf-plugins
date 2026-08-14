---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

JARVIS OS — an AI-native Linux distribution built on an Arch/CachyOS base. The AI is a first-class kernel citizen via a custom character device (`/dev/jarvis`). Three layers:

1. **`linux-jarvisos/`** — kernel submodule with JARVIS drivers (`drivers/jarvis/`)
2. **`Project-JARVIS/`** — AI daemon submodule (dispatch + dmcp + contextor)
3. **`iso-build-scripts/`** — ISO build pipeline that layers everything onto a base Arch ISO

## Design Principles

**Auto-install missing dependencies — never warn and bail.** Any package required by a build step, installer mode, or runtime must be installed automatically if not present. Do not warn the user that a tool is missing and exit — install it first, then proceed. This applies everywhere:
- `jarvis-install.sh --overlay` / `--install-packages` (repo-root script, doubles as live-ISO TUI and chroot dependency installer) — installs `dialog`, `base-devel`, `bc`, `flex`, `bison`, `openssl`, `libelf`, `pahole` at the top of `install_packages_mode()` before anything else runs
- `iso-build-scripts/00-install-prereq.sh` (`make prereq`) — installs all ISO build tools + kernel build tools for all supported distros (Arch, Fedora, Ubuntu, openSUSE)
- Any new feature that needs a host tool: add it to both `jarvis-install.sh`'s dep block and `00-install-prereq.sh`

## Build Config

`build.config` at the **project root** (not `iso-build-scripts/`) is sourced by all scripts and the Makefile:
- `PROJECT_ROOT` — auto-detected via `dirname`; override only if needed
- `ISO_FILE` — source ISO filename (currently `archlinux-x86_64.iso`); place file in `build-deps/`
- `BUILD_DIR`, `BUILD_DEPS_DIR` — relative to `PROJECT_ROOT`

**Arch-based host required** for the kernel build step — `makepkg` and `pacman` are mandatory.

## Build Commands

All ISO build commands run from `iso-build-scripts/`:

```bash
cd iso-build-scripts

make prereq          # Install host build tools (detects Arch/Fedora/Ubuntu/openSUSE)
make all             # Full build: steps 1–7 + 3b (prereq must be run first)
make rest            # Resume interrupted build (skips completed steps)
make status          # Show which steps are done
make clean           # Wipe build/iso-extract/ and build/iso-rootfs/

make step1           # Extract base ISO
make step2           # Unsquash rootfs
make step3           # Install KDE Plasma Wayland + all packages into rootfs
make step3b          # Build linux-jarvisos kernel (bottleneck: 20–60 min first run)
make step4           # Install Project-JARVIS daemon
make step5           # Install TUI installer (jarvis-install)
make step6           # Repack SquashFS
make step7           # Assemble final ISO

make JOBS=8 step3b   # Parallel kernel build (default: nproc)
```

Kernel-only build (host install or package-only):
```bash
bash iso-build-scripts/03b-build-kernel.sh --host-install   # Build + install on running system
bash iso-build-scripts/03b-build-kernel.sh                  # Build packages only → build/kernel-pkg/
SKIP_KERNEL_BUILD=1 bash iso-build-scripts/03b-build-kernel.sh --host-install  # Skip recompile
```

Test in QEMU:
```bash
./iso-build-scripts/booter.sh         # UEFI boot
./iso-build-scripts/booter.sh --bios  # BIOS boot
```

Standalone agent (no ISO needed):
```bash
./test-jarvis-ollama.sh                           # Auto-setup + launch
JARVIS_MODEL=qwen3:8b ./test-jarvis-ollama.sh     # Force model
OLLAMA_URL=http://192.168.1.10:11434 ./test-jarvis-ollama.sh
```

## Architecture

```
LLM (Ollama) → dispatch (tool routing) → dmcp (MCP server registry)
                                        ↓
                          JARVIS Policy Gate (SAFE/ELEVATED/DANGEROUS/FORBIDDEN)
                                        ↓
                              Shell execution (PTY)
                                        ↓
                          linux-jarvisos kernel (/dev/jarvis)
                          /sys/class/misc/jarvis/sysmon/   ← hw metrics
                          /sys/class/misc/jarvis/policy/   ← policy table
```

### Kernel Drivers (`linux-jarvisos/drivers/jarvis/`)

| File | Purpose |
|------|---------|
| `jarvis_core.c` | `/dev/jarvis` misc device + query ring buffer |
| `jarvis_sysmon.c` | CPU/memory/thermal via ioctl and sysfs |
| `jarvis_policy.c` | 4-tier action policy engine with rate limiting |
| `jarvis_keys.c` | API key storage in Linux kernel keyring |
| `jarvis_dibs.c` | Zero-copy DIBS buffer for large inference payloads |
| `include/uapi/linux/jarvis.h` | Userspace API (ioctls, structs, enums) |

PKGBUILD at `packages/linux-jarvisos/PKGBUILD` — reads `KERNEL_SRC` env var pointing at the submodule.

### Project-JARVIS Submodule (`Project-JARVIS/`)

- **`dispatch/`** — Rust crate; routes user intent to MCP tools (keyword fallback, embedding-first when available)
- **`dmcp/`** — Rust crate; MCP server registry and lifecycle manager
- **`contextor/`** — Rust crate; context/memory layer


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JarvisOSLinux/jarvisos](https://github.com/JarvisOSLinux/jarvisos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->

---
trigger: always_on
description: **axiom-ebpf** is a hobby x86-64 operating system kernel written in Rust. This is a bare-metal OS project that boots using the Limine bootloader and runs on QEMU. The project consists of ~124 Rust source files organized into a kernel and userspace components.
---

# axiom-ebpf - Copilot Coding Agent Instructions

## Project Overview

**axiom-ebpf** is a hobby x86-64 operating system kernel written in Rust. This is a bare-metal OS project that boots using the Limine bootloader and runs on QEMU. The project consists of ~124 Rust source files organized into a kernel and userspace components.

**Project Type:** Operating System Kernel (Bare Metal)
**Primary Language:** Rust (Nightly)
**Target Architecture:** x86_64-unknown-none
**Bootloader:** Limine v9.x
**Build System:** Cargo with custom build.rs scripts
**Repository Size:** Medium (~124 .rs files, 18 .toml files)

## Critical Build Requirements

**System Dependencies:** Install `xorriso` before attempting full builds: `sudo apt install -y xorriso`

**Rust Toolchain:** Nightly channel with components: rustfmt, clippy, llvm-tools-preview, rust-src, miri. Target: x86_64-unknown-none. All configured in `rust-toolchain.toml` and auto-installed.

## Build Process

**WARNING:** Full builds may fail with SSL certificate errors when downloading OVMF firmware. This is why we use `--lib` for validation.

### Build Commands

**DO NOT use `cargo build` or `cargo build --release` on the root crate** - it will fail with the SSL certificate issue mentioned above.

**To build only library crates (kernel subsystems) - RECOMMENDED for validation:**
```bash
cargo build --workspace --lib
```

**To build in release mode:**
```bash
cargo build --workspace --lib --release
```

**Note:** The `kernel`, `init`, and `muffinos` binaries are bare-metal targets that require special build process via the root build.rs. Building these directly will fail. Use `--lib` to build only the library crates which can be built normally.

**Build time:** Expect 1-3 minutes for a clean library build, 2-5 minutes if binaries are included.

The main `muffinos` binary is a runner that builds the kernel, creates a bootable ISO, and launches QEMU. It's primarily used for running the OS, not for code validation.

### Build Artifacts

Key artifacts in `target/`: Limine bootloader, OVMF firmware, `muffin.iso` (bootable), `disk.img` (ext2 filesystem), kernel binary.

## Testing

### Running Tests

**The kernel crate cannot run standard tests** because it uses a custom linker script for bare-metal targets. Tests will fail with TLS linker errors if you try to run them on the kernel library.

**To run tests on individual workspace crates (RECOMMENDED):**
```bash
cargo test -p kernel_abi
cargo test -p kernel_vfs
cargo test -p kernel_physical_memory
# etc.
```

**DO NOT use `cargo test --workspace --lib`** - it will attempt to test the kernel library with the custom linker and fail.

**To run tests on all kernel subsystem crates:**
```bash
for crate in kernel_abi kernel_devfs kernel_device kernel_elfloader kernel_memapi kernel_pci kernel_physical_memory kernel_syscall kernel_vfs kernel_virtual_memory; do
    cargo test -p $crate
done
```

Note: Most kernel crates have no tests (0 tests run is normal). The kernel itself cannot have unit tests due to its bare-metal nature.

### Miri Tests

The CI runs Miri tests on kernel crates to check for undefined behavior:

```bash
cargo miri setup
cargo miri test -p kernel_abi
cargo miri test -p kernel_vfs
# etc.
```

Miri tests are run separately for each kernel crate. See `.github/workflows/build.yml` for the complete list.

**Miri test time:** ~30 seconds per crate

## Linting and Formatting

### Code Formatting

The project uses rustfmt with custom configuration in `rustfmt.toml`:
```toml
imports_granularity = "Module"
group_imports = "StdExternalCrate"
```

**To check formatting:**
```bash
cargo fmt -- --check
```

**To apply formatting:**
```bash
cargo fmt
```

**Formatting time:** ~1-5 seconds

### Clippy Linting

**ALWAYS run clippy on workspace libraries:**
```bash
cargo clippy --workspace --lib -- -D clippy::all
```

You can also exclude the main binary explicitly:
```bash
cargo clippy --workspace --exclude muffinos -- -D clippy::all
```

**DO NOT run `cargo clippy` without filters** - it will try to build bare-metal binaries and fail.

The CI enforces all clippy warnings as errors (`-D clippy::all`), so you must fix all clippy warnings.

**Clippy time:** ~10-30 seconds for incremental checks, ~2-5 minutes for clean builds

### Known Warnings

The kernel has intentional dead code warnings for unused fields (`physical_frames`, `node` in memory region structs). These are expected.

## Project Architecture

### Directory Structure

```
├── .github/workflows/build.yml  # CI/CD pipeline
├── kernel/                      # Main kernel crate
│   ├── crates/                 # 10 kernel subsystem crates (abi, devfs, device, elfloader, 
│   │                           #   memapi, pci, physical_memory, syscall, vfs, virtual_memory)
│   ├── src/                    # Kernel source (arch/, driver/, file/, mcore/, syscall/)
│   ├── linker-x86_64.ld        # Custom linker script
│   └── Cargo.toml
├── userspace/                  # file_structure, init, minilib
├── src/main.rs                 # QEMU runner

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pro-utkarshM/axiomOS](https://github.com/pro-utkarshM/axiomOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

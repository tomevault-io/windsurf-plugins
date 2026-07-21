---
trigger: always_on
description: LK is a small, SMP-aware embedded OS kernel designed for supervisor mode on diverse 32/64-bit architectures. It's used extensively in embedded systems, including Android bootloaders. Written primarily in C and assembly, with limited C++ (no STL, no exceptions).
---

# LK Kernel Development Guide

LK is a small, SMP-aware embedded OS kernel designed for supervisor mode on diverse 32/64-bit architectures. It's used extensively in embedded systems, including Android bootloaders. Written primarily in C and assembly, with limited C++ (no STL, no exceptions).

## Architecture Overview

### Hierarchical Build System

LK uses a 4-layer modular build system:

1. **Project** (`project/*.mk`) - Top-level configuration defining which modules to include
   - Example: `project/qemu-virt-arm64-test.mk` includes shell, filesystem, networking modules
   - Projects include other project fragments: `include project/virtual/test.mk`

2. **Target** (`target/*.mk`) - Board-specific configuration combining platform + hardware details
   - Defines memory layout: `MEMBASE`, `MEMSIZE`, `KERNEL_BASE`
   - GPIO configs, peripheral addresses for specific boards

3. **Platform** (`platform/*/`) - SOC/system-level support (qemu-virt, stm32f4xx, etc.)
   - Hardware initialization, device tree handling, platform-specific drivers

4. **Architecture** (`arch/*/`) - CPU-specific low-level code (arm64, riscv, x86, etc.)
   - MMU setup, exception handling, context switching, atomic ops

### Module System Pattern

Every component is a module with a `rules.mk` file:

```make
LOCAL_DIR := $(GET_LOCAL_DIR)
MODULE := $(LOCAL_DIR)

MODULE_SRCS += \
    $(LOCAL_DIR)/foo.c \
    $(LOCAL_DIR)/bar.c

MODULE_DEPS += \
    lib/libc \
    kernel

MODULE_OPTIONS := extra_warnings  # Enables stricter compiler flags

include make/module.mk
```

**Key points:**

- `MODULE := $(LOCAL_DIR)` is required - sets module name to directory path
- `MODULE_DEPS` creates dependency tree, automatically included in build
- `MODULE_OPTIONS`: `extra_warnings` adds strict checks, `float` enables FP compilation
- Module include paths auto-added: `$(MODULE)/include/` becomes available globally
- Always use `$(LOCAL_DIR)` prefix for source paths
- Must `include make/module.mk` at end of `rules.mk` to finalize the module definition
- All MODULE_* variables are cleared after inclusion, preventing leakage between modules
- Modules are built as separate ELF .o files and linked into the final kernel image
- Modules commonly export their include name space that matches the name of the module.
  - ie. lib/foo will export the include path `lib/foo.h` with any additioal headers under `lib/foo/`.

## Critical Build Patterns

### Building Projects

```bash
# Build specific project (creates build-<project>/ directory)
make qemu-virt-arm64-test

# Or just use project name as target
make PROJECT=qemu-virt-arm64-test

# Override heap implementation
make qemu-virt-arm64-test LK_HEAP_IMPLEMENTATION=cmpctmalloc

# Debug builds (default DEBUG=2, set to 0 for release)
make qemu-virt-arm64-test DEBUG=0

# Clean specific project.
# Add a 'clean' target to the end of the command line.
# Either select the project name or use the PROJECT variable.
make qemu-virt-arm64-test clean

# Clean everything
make spotless

# Build all projects (for CI/verification)
scripts/buildall -q -e -r  # quiet, warnings-as-errors, release builds

Output will be written to buildall.log. To run the build with full output during the build, omit the -q flag.
```

The file local.mk is silently included if it exists in the root directory. Additional variables can be defined in this file to customize the build instead of needing to pass them on the command line.

### Build output

The build output will be written to the `build-<project>/` directory, where `<project>`
is the name of the project being built.

Build artifacts include object files, libraries, executables, and generated headers:

- srcfiles.txt - list of source files used in the build
- include_paths.txt - list of include paths used in the build
- buildid.h - generated build ID header file
- config.h - generated config header included by all source files in the system.
- lk.bin - flattened kernel binary
- lk.elf - pre-flattened kernel image, ELF format with debugging information
- lk.elf.lst - disassembly of the kernel image
- lk.elf.debug.lst - disassembly of the kernel image with debug information
- lk.elf.map - linker map file
- lk.elf.sym - symbol table for the kernel image
- lk.elf.size - size information for the kernel image
- lk.elf.sym - symbol table for the kernel image
- lk.elf.sym.sorted - sorted symbol table for the kernel image, sorted by address
- lk.elf.dump - equivalent of objdump -x on the lk.elf file

Each module is linked into an ELF intermediate file named `<module>.mod.o` via ld -r.
The module and object file paths follow the same path structure as the source files.

### Running Tests

Scripts in `scripts/` launch QEMU with appropriate flags:

```bash
# ARM64 (4KB pages)
scripts/do-qemuarm -6

# ARM64 with 64KB pages
scripts/do-qemuarm -6 -P 64k

# ARM64 with KVM/HVF acceleration (only if on ARM64 host)
scripts/do-qemuarm -6 -k

# RISC-V 32-bit in machine mode
scripts/do-qemuriscv

# RISC-V 64-bit in supervisor mode and paging
scripts/do-qemuriscv -6S

# x86-32
scripts/do-qemux86

# x86-64

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [littlekernel/lk](https://github.com/littlekernel/lk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

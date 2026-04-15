---
trigger: always_on
description: **ALWAYS FOLLOW THESE INSTRUCTIONS FIRST.** Only fallback to additional search and context gathering if the information in these instructions is incomplete or found to be in error.
---

# Speculative Execution Repository

**ALWAYS FOLLOW THESE INSTRUCTIONS FIRST.** Only fallback to additional search and context gathering if the information in these instructions is incomplete or found to be in error.

This repository contains educational proof-of-concept (PoC) implementations of the Meltdown (CVE-2017-5754) and Spectre (CVE-2017-5753) CPU vulnerabilities. It includes reference exploits from published research (Jann Horn et al., paboldin, rootkea) alongside original cross-process Spectre variants and supporting test utilities, all written in C99 for x86/x86-64 Linux.

## Quick Reference

**Essential Commands:**
- `mkdir build && cd build && cmake .. && make` - Build all targets (~10-30s depending on CPU)
- `./build/jan_horn` - Run Spectre v1 PoC (Jann Horn)
- `./build/cache_time` - Run cache timing test
- `cd Meltdown/paboldin && make -f Makefile.txt && ./run.sh` - Build and run Meltdown standalone
- `gcc --version && cmake --version` - Check prerequisites

**CMake Build Targets:**

| Target | Source | Description |
|--------|--------|-------------|
| `jan_horn` | `Spectre/jann_horn_et_al/script.c` | Canonical Spectre v1 PoC |
| `rootkea` | `Spectre/rootkea/script.c` | Spectre v1 with custom secret |
| `paboldin` | `Meltdown/paboldin/script.c` | Meltdown kernel memory reader |
| `victim` | `Spectre/rios0rios0/victim.c` | Cross-process victim (single-shot) |
| `victim2` | `Spectre/rios0rios0/victim2.c` | Cross-process victim (looping) |
| `attacker` | `Spectre/rios0rios0/attacker.c` | Cross-process Spectre attacker |
| `cache_time` | `Tests/CacheTime.c` | Cache hit/miss timing |
| `kernel_table` | `Tests/KernelTable.c` | Fork/virtual memory test |
| `virt_address` | `Tests/VirtAddress.c` | Virtual-to-physical translation |
| `size_of` | `Tests/SizeOf.c` | sizeof vs strlen demo |

**Technology Stack:** C99 (`_GNU_SOURCE`), inline x86/x86-64 AT&T assembly, `rdtscp`/`rdtsc`, `clflush` (`_mm_clflush`), `sigaction`, `sched_setaffinity`, CMake 3.13+

**No CI/CD pipelines and no automated tests** -- this is a research/educational project.

## Working Effectively

### Prerequisites

- **GCC 7+** with x86 intrinsics (`_mm_clflush`, `__rdtscp`, `__builtin_ia32_*`)
- **CMake 3.13+**
- **GNU Make** or **Ninja**
- **Linux x86-64** (required for `/proc/kallsyms`, `/proc/pagemap`, inline AT&T assembly)

### Bootstrap and Setup

```bash
git clone https://github.com/rios0rios0/speculative-execution.git
cd speculative-execution
mkdir build && cd build
cmake ..
make
```

### Core Commands That Work

```bash
# Verify prerequisites
gcc --version                    # GCC 7+
cmake --version                  # CMake 3.13+

# Build all targets
mkdir build && cd build
cmake ..                         # Configure (~1-2s)
make                             # Build all (~10-30s)

# Run Spectre PoC (Jann Horn)
./build/jan_horn                 # Reads "The Magic Words are Squeamish Ossifrage."

# Run cache timing test
./build/cache_time               # Shows cached (~5-30 cycles) vs uncached (~100-300 cycles)

# Run Meltdown standalone build
cd Meltdown/paboldin
make -f Makefile.txt             # Standalone build with -O2 -msse2
./run.sh                         # Reports VULNERABLE or NOT VULNERABLE

# Run cross-process Spectre (two terminals)
./build/victim2                  # Terminal 1: prints PID and virtual address
./build/attacker <pid> <vaddr>   # Terminal 2: run attacker with victim's output
```

### Standalone Meltdown Build (Alternative)

```bash
cd Meltdown/paboldin
# Optional: auto-detect rdtscp support
bash detect_rdtscp.sh            # Generates rdtscp.h
make -f Makefile.txt             # Produces ./script binary
./run.sh                         # Full vulnerability test
```

## Repository Structure

```
speculative-execution/
├── CMakeLists.txt                          # Build configuration (10 targets, C99)
├── README.md                               # Project documentation
├── CONTRIBUTING.md                         # Development guidelines
├── LICENSE                                 # MIT License
│
├── Meltdown/
│   └── paboldin/
│       ├── script.c                        # Meltdown PoC: reads kernel memory via speculative execution
│       ├── Makefile.txt                    # Standalone Makefile with -O2 -msse2 flags
│       ├── run.sh                          # Locates linux_proc_banner and runs the exploit
│       └── detect_rdtscp.sh                # Generates rdtscp.h with rdtscp or rdtsc+mfence
│
├── Spectre/
│   ├── jann_horn_et_al/
│   │   └── script.c                        # Canonical Spectre v1 PoC (bounds check bypass)
│   ├── rootkea/
│   │   └── script.c                        # Spectre v1 variant with custom secret string
│   └── rios0rios0/
│       ├── attacker.c                      # Cross-process Spectre attacker via /proc/pagemap
│       ├── victim.c                        # Victim process (single-shot, exits after one call)
│       └── victim2.c                       # Victim process (loops, prints PID and vaddr)
│
└── Tests/
    ├── CacheTime.c                         # Cache hit/miss timing across 10 cache lines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rios0rios0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

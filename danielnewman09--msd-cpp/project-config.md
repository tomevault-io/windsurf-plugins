---
trigger: always_on
description: Generates API documentation from source code comments. Automatically enabled when Doxygen is installed.
---

# MSD-CPP Project Guide

> This document provides high-level context for AI assistants and developers working on the MSD-CPP repository.

## Project Overview

MSD-CPP (Multi-Spacecraft Dynamics) is a C++ project for spacecraft dynamics simulation and visualization. The core libraries are located in the [`msd/`](msd/) directory.

**For detailed library architecture and component documentation, see [`msd/CLAUDE.md`](msd/CLAUDE.md).**

---

## Repository Structure

```
MSD-CPP/
├── msd/                      # Core libraries (see msd/CLAUDE.md)
│   ├── msd-transfer/         # Database transfer objects (DTOs)
│   ├── msd-assets/           # Asset management and geometry factories
│   ├── msd-sim/              # Physics simulation engine
│   ├── msd-gui/              # GPU-accelerated 3D rendering
│   ├── msd-exe/              # Main executable
│   └── msd-asset-gen/        # Asset generation tool
│
├── docs/                     # Documentation
│   ├── designs/              # Feature design documents and PlantUML diagrams
│   ├── msd/                  # Library-specific diagrams
│   ├── workflows/            # Development workflow documentation
│   ├── benchmarking.md       # Benchmarking guide
│   └── profiling.md          # Profiling guide (macOS)
│
├── tickets/                  # Feature tickets (see Ticketing System below)
├── prototypes/               # Prototype code for design validation
│
├── scripts/                  # Tooling and automation
│   ├── generate_record_layers.py  # Auto-generates pybind11 bindings and Pydantic models from C++ records
│   └── traceability/         # Design decision traceability (see scripts/traceability/README.md)
│
├── analysis/                 # Performance analysis infrastructure (see analysis/CLAUDE.md)
│   ├── scripts/              # Benchmarking and profiling scripts
│   ├── benchmark_baselines/  # Golden baselines for benchmark comparison
│   └── profile_baselines/    # Golden baselines for profiling comparison
│
├── build/                    # CMake build output (gitignored)
├── conan/                    # Conan package manager configuration
├── test/                     # Integration test resources
│
├── CMakeLists.txt            # Root CMake configuration
├── CMakeUserPresets.json     # Build presets for component builds
├── conanfile.py              # Conan dependency specification
└── CLAUDE.md                 # This file
```

---

## Ticketing System

The project uses a ticket-based workflow for feature development. Tickets live in [`tickets/`](tickets/) and follow a structured format.

### Ticket Lifecycle

1. **New** — Ticket created, requirements defined
2. **Design** — Architectural design in `docs/designs/{ticket-name}/`
3. **Design Review** — Design reviewed and approved
4. **Prototype** — Validation code in `prototypes/{ticket-name}/`
5. **Implementation** — Production code written
6. **Implementation Review** — Code reviewed
7. **Documentation** — CLAUDE.md and diagrams updated
8. **Complete** — Ticket closed

### Ticket Naming Convention

Tickets use a numeric prefix for ordering: `NNNN_descriptive_name.md`

Examples:
- `0011_add_google_benchmark.md`
- `0015_profiling_trace_parser.md`

### Design Documents

Each ticket with architectural changes has a design folder:
```
docs/designs/{ticket-name}/
├── design.md                 # Architectural design document
├── {ticket-name}.puml        # PlantUML diagram
└── prototype-results.md      # Prototype findings (if applicable)
```

---

## Build & Configuration

### Build Requirements
- C++ Standard: C++20
- Compiler: GCC 11+, Clang 14+, or MSVC 2019+
- Build System: CMake 3.15+ with Conan 2.x package manager
- Dependencies: Managed via Conan (see `conanfile.py`)

### Build Process

This project uses **Conan** for dependency management and **CMake presets** for build configuration.

#### Prerequisites: Install Dependencies with Conan

**IMPORTANT**: Before building, you must run Conan to install dependencies and generate CMake configuration files:

```bash
# For Debug build
conan install . --build=missing -s build_type=Debug

# For Release build
conan install . --build=missing -s build_type=Release

# For Debug build with code coverage enabled
conan install . --build=missing -s build_type=Debug -o "&:enable_coverage=True"

# For Release build with benchmarks enabled
conan install . --build=missing -s build_type=Release -o "&:enable_benchmarks=True"
```

This generates CMake preset files in `build/Debug/generators/CMakePresets.json` and `build/Release/generators/CMakePresets.json` which are referenced by [`CMakeUserPresets.json`](CMakeUserPresets.json).

#### Building the Entire Project

After running `conan install`, configure and build:

```bash
# Configure with Debug preset
cmake --preset conan-debug

# Build everything (Debug)
cmake --build --preset conan-debug

# Or for Release
cmake --preset conan-release
cmake --build --preset conan-release
```

#### Building Specific Components

The project provides component-specific build presets in [`CMakeUserPresets.json`](CMakeUserPresets.json) to build individual libraries without building the entire project:

**Debug builds:**
```bash
cmake --build --preset debug-utils-only      # msd-utils library + tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielnewman09) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

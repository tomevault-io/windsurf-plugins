---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

swift-netcdf is a Swift wrapper for the NetCDF (Network Common Data Form) C library. The project builds netCDF-c from source as a git submodule and exposes it to Swift through a C module wrapper.

## Architecture

### Three-Layer Structure

1. **netcdf-c (C library)**: Git submodule at `Sources/CNetCDF/netcdf-c`
   - Official Unidata netCDF C library
   - Built from source using CMake
   - Configured for minimal build (netCDF-3 only, no HDF5/DAP/NCZarr)

2. **CNetCDF (C module)**: Swift Package Manager C target at `Sources/CNetCDF`
   - Module map at `Sources/CNetCDF/module.modulemap` defines the C module
   - Built library artifacts in `Sources/CNetCDF/lib/` (generated, not in git)
   - Exposes C headers to Swift

3. **NetCDF (Swift wrapper)**: Swift library at `Sources/NetCDF/`
   - Public Swift API wrapping C functions
   - Type-safe Swift interfaces (NetCDFFile, NetCDFMode, NetCDFError)
   - Manages resource lifecycle and error handling

### Key Design Decisions

- **Source build required**: The netCDF C library must be built before Swift package can compile
- **Static linking**: Uses libnetcdf.a (static library) to avoid runtime dependencies
- **Minimal configuration**: netCDF built without HDF5, DAP, NCZarr to reduce complexity
- **Module map approach**: Uses modulemap to expose C library to Swift (not systemLibrary target)
- **Type-safe API**: Separate AccessMode and CreationMode types prevent API misuse
- **Swift-like interface**: Uses computed properties, URL-based APIs, and proper error handling

## Build Process

### Initial Setup (First Time Only)

```bash
# 1. Initialize git submodules
git submodule update --init --recursive

# 2. Build netcdf-c library
./scripts/build-netcdf.sh
```

This creates:
- `Sources/CNetCDF/lib/include/` - C headers
- `Sources/CNetCDF/lib/lib/libnetcdf.a` - Static library
- `.build/netcdf-build/` - CMake build artifacts (can be deleted)

### Regular Development

```bash
# Build Swift package
swift build

# Run tests
swift test

# Build in verbose mode (for debugging)
swift build -v
```

### Rebuild netcdf-c

If you need to rebuild the C library (e.g., after updating submodule or platform version):

```bash
# Clean previous build
rm -rf .build/netcdf-build Sources/CNetCDF/lib

# Rebuild
./scripts/build-netcdf.sh
```

**Important**: The build script automatically sets `CMAKE_OSX_DEPLOYMENT_TARGET` to match the platform requirements in Package.swift. If you update the platform versions (e.g., from macOS 15 to macOS 26), you **must** rebuild the NetCDF C library to avoid runtime crashes.

## Package.swift Configuration

The CNetCDF target in Package.swift:
- Uses `.target()` (NOT `.systemLibrary()`)
- `publicHeadersPath: "include"` points to umbrella header directory
- `sources: ["dummy.c"]` - includes a dummy C file for Xcode compatibility
  - The actual NetCDF library is pre-built and linked statically
  - `dummy.c` satisfies Xcode's requirement that targets have at least one source file
  - Command-line builds work with or without this file
- `cSettings: [.headerSearchPath("lib/include")]` - finds built headers
- `linkerSettings` links against pre-built `libnetcdf.a`
- Must exclude submodule directory and build artifacts with `exclude`

## Common Issues

### Runtime Crash in NC_create() or other NetCDF functions

**Symptom**: Tests crash inside NetCDF C library functions (like `NC_create()`)

**Cause**: Deployment target mismatch. The NetCDF library was built for a different macOS version than what the Swift package expects.

**Solution**:
1. Clean and rebuild the NetCDF C library:
   ```bash
   rm -rf .build/netcdf-build Sources/CNetCDF/lib
   ./scripts/build-netcdf.sh
   ```
2. Rebuild the Swift project:
   ```bash
   swift build
   ```

The build script automatically sets the correct deployment target (26.0) via `-DCMAKE_OSX_DEPLOYMENT_TARGET=26.0`.

### Xcode Build: "Build input file cannot be found: CNetCDF.o"

If building in Xcode fails with this error:
- Ensure `Sources/CNetCDF/dummy.c` exists
- Verify `Package.swift` includes `sources: ["dummy.c"]` in CNetCDF target
- This dummy file is required for Xcode but not for command-line builds

The issue occurs because Xcode requires at least one source file per target, even for targets that only link pre-built libraries.

### Missing netcdf_filter_hdf5_build.h

The built netCDF library may not install all headers referenced in the module map. If you encounter missing header errors:
- Check which headers are actually installed in `Sources/CNetCDF/lib/include/`
- Update `Sources/CNetCDF/module.modulemap` to only include available headers
- May need to adjust CMake build options in `scripts/build-netcdf.sh`

### Module CNetCDF not found

Ensure:
1. `./scripts/build-netcdf.sh` has been run successfully
2. `Sources/CNetCDF/lib/include/netcdf.h` exists
3. `Sources/CNetCDF/module.modulemap` exists and references correct header paths

### Linker errors

- Verify `Sources/CNetCDF/lib/lib/libnetcdf.a` exists
- Check that linker flags in Package.swift use correct relative paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/1amageek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

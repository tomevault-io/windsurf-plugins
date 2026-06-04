---
trigger: always_on
description: Pible is a HolyC to BPF compiler written in Zig that transforms HolyC programs into BPF bytecode for kernel-space execution. This compiler bridges Terry Davis's HolyC with Linux BPF systems.
---

# Pible - HolyC to BPF Compiler

Pible is a HolyC to BPF compiler written in Zig that transforms HolyC programs into BPF bytecode for kernel-space execution. This compiler bridges Terry Davis's HolyC with Linux BPF systems.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

**IMPORTANT**: All timing estimates and command validations in these instructions have been tested and verified. Build times are much faster than typical Zig projects due to no external dependencies.

## Rules for This Project

### Zig Version Requirements
- **ALWAYS use Zig 0.16.x or later** for this project
- The project requires Zig 0.16.x minimum for proper build system and ArrayList API compatibility
- Do not use older Zig versions (0.15.x or earlier) as they have incompatible APIs:
  - ArrayList.init() vs ArrayList{} initialization
  - append() requiring allocator parameter 
  - deinit() requiring allocator parameter
  - writer() requiring allocator parameter
  - toOwnedSlice() requiring allocator parameter
  - .root_source_file vs .root_module build API changes
- When updating Zig version requirements, always test the build to ensure compatibility

## Working Effectively

### What This Compiler Does
Pible transforms HolyC programs into BPF (Berkeley Packet Filter) bytecode that can run in Linux kernel space. The compilation pipeline:
1. **Lexer** → Tokenizes HolyC source code
2. **Parser** → Builds Abstract Syntax Tree (AST)  
3. **CodeGen** → Generates BPF instructions using zbpf library
4. **Output** → Produces .bpf files containing BPF bytecode

### Prerequisites and Setup
- **CRITICAL**: Install Zig programming language (version 0.16.x or later):
  - **Primary method**: Download from https://ziglang.org/builds/ (latest 0.16.x development build)
    ```bash
    cd /tmp
    # Get latest development build (0.16.x) - check https://ziglang.org/download/index.json for current version
    wget https://ziglang.org/builds/zig-x86_64-linux-0.16.0-dev.13+1594c8055.tar.xz
    tar -xf zig-x86_64-linux-0.16.0-dev.13+1594c8055.tar.xz
    export PATH=/tmp/zig-x86_64-linux-0.16.0-dev.13+1594c8055:$PATH
    ```
  - **Alternative method**: Via package manager (if available):
    ```bash
    # Ubuntu/Debian (if available)
    sudo apt install zig
    # Or via snap
    sudo snap install zig --classic
    ```
  - **Verify installation**: `zig version` (should show 0.16.x or later)
  - **IMPORTANT NOTE**: If the specific Zig build URL above returns 404, check https://ziglang.org/download/index.json for the current development build URL and update accordingly.

### Build and Test Commands
- **TIMING**: Build commands are very fast in this project (~6 seconds). Tests are even faster (<1 second).
- **NO EXTERNAL DEPENDENCIES**: This project has no external dependencies to fetch during build.
- **TIMEOUTS**: Use 60+ second timeouts for build commands as a safety buffer, though builds typically complete in seconds.

**Bootstrap and build the repository:**
```bash
cd /path/to/holyBPF-zig
zig build                    # Typically completes in ~6 seconds. No dependency fetching required.
```

**Run the test suite:**
```bash
zig build test              # Typically completes in <1 second. Runs all tests in tests/ directory.
```

**Build specific examples:**
```bash
zig build hello-world       # Build the hello-world example BPF program (~0.03 seconds)
zig build escrow            # Build the escrow example BPF program (~0.03 seconds)
# Note: solana-token example currently has parsing issues - this is a known limitation
```

**Compile HolyC programs:**
```bash
# Compile a HolyC file to BPF bytecode
./zig-out/bin/pible examples/hello-world/src/main.hc
# This produces main.hc.bpf containing the BPF bytecode

# Verify the output is valid BPF bytecode
file examples/hello-world/src/main.hc.bpf
hexdump -C examples/hello-world/src/main.hc.bpf | head -5
```

**Optional BPF testing (if bpf tools available):**
```bash
# These commands may not work in all environments - BPF tools are not typically available
bpf-cli verify examples/hello-world/src/main.hc.bpf    # Verify BPF program (if bpf-cli installed)
bpf-cli run examples/hello-world/src/main.hc.bpf       # Run BPF program (if bpf-cli installed)
```

## Validation

### Manual Testing Scenarios
After making changes, ALWAYS run through these validation scenarios:

1. **Basic Compilation Test:**
   ```bash
   # Test compiler on hello-world example
   ./zig-out/bin/pible examples/hello-world/src/main.hc
   # Verify main.hc.bpf file is created and non-empty
   ls -la examples/hello-world/src/main.hc.bpf
   file examples/hello-world/src/main.hc.bpf  # Should show binary data
   ```

2. **Error Handling Test:**
   ```bash
   # Create a file with invalid HolyC syntax
   echo "U0 broken() { invalid_syntax" > /tmp/broken.hc
   # Verify compiler reports appropriate error (should fail gracefully)
   ./zig-out/bin/pible /tmp/broken.hc
   ```

3. **Test Suite Validation:**
   ```bash
   # Always run full test suite after changes - completes very quickly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pibleos/holyBPF-zig](https://github.com/pibleos/holyBPF-zig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->

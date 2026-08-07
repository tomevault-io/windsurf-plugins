---
trigger: always_on
description: This is the `meow` repository. `meow` is a drop-in replacement for Node.js, an ultra-fast package manager, a testing framework, and a unified toolchain, delivered as a single Rust binary.
---

# 🐾 meow: Agent Architecture & Contribution Guidelines

This is the `meow` repository. `meow` is a drop-in replacement for Node.js, an ultra-fast package manager, a testing framework, and a unified toolchain, delivered as a single Rust binary.

It is built on the **"Floof & Teeth"** duality:
*   **The Floof:** Best-in-class, adorable, empathetic Terminal UX (Bento boxes, paw spinners, high-fidelity diagnostics).
*   **The Teeth:** Ruthless, zero-overhead systems engineering (Oxc, V8 snapshots, APFS cloning, Tokio parallelism).

This document defines the physical laws of the `meow` architecture. If you are an AI agent or contributor writing code for this repository, you must obey these invariants.

---

## 1. The Zero-Compromise Engineering Philosophy

Before you generate a single line of code, you must pass your proposed solution through this internal checklist:

1. **Does this create technical debt, or is it a shortcut just to get it to work now?** 
   If yes, it is an immediate violation. We do not ship half-baked features. If doing it the "right" way (e.g., integrating native Rolldown vs. doing naive string concatenation) takes longer, we take the longer path. 
2. **Does this compromise correctness to make a test pass?** 
   If yes, it is an immediate violation. We do not write hacks to satisfy broken tests. If a test is failing, fix the underlying JavaScript or Rust logic. We test against reality.
3. **Does this sacrifice fundamentals to make a benchmark look good?** 
   If yes, it is an immediate violation. We do not drop cryptographic supply-chain checks (SHA-512) to win cold-install speed benchmarks. We win benchmarks through superior systems engineering (SIMD, OS thread pooling, kernel syscalls).
4. **The Prime Directive: Amputate, Do Not Medicate.** 
   If a legacy system, upstream crate, or abstraction is fundamentally flawed, do not patch over it with `if` statements or `// TODO` comments. Rip out the root cause and replace it with a structurally sound, Rust-native implementation.

---

## 2. Architectural Invariants

### The "Parse-Once" Oxc Pipeline
*   `meow` uses `oxc` for all parsing, semantic analysis, TypeScript type-stripping, and JSX transformation. 
*   **SWC and `deno_ast` are strictly forbidden in this codebase.** If you need to transform or parse code, you use `oxc_allocator`, `oxc_parser`, `oxc_transformer`, and `oxc_codegen`. 
*   We do not emit sourcemaps or downlevel modern JavaScript. We strip TypeScript annotations in-place.

### Package Management & Materialization
*   **No Symlinks for Packages:** We do not use symlinks to materialize `node_modules` packages, as this breaks V8 and Vite's `fs.realpath` resolution.
*   **The APFS / Hardlink Strategy:** Packages are materialized into project-local `node_modules` using the macOS `clonefile(2)` kernel syscall for O(1) cloning. On Linux/Windows, we fall back to recursive hardlinking. 
*   Dependency *edges* (the pointers inside a package's `node_modules` linking to another package) remain links, not copies: Unix uses symlinks; Windows uses NTFS directory junctions because directory symlinks require privileges in CI. Do not replace Windows edges with deep copies — that hides graph bugs and bloats node_modules.

### Cryptography and Network I/O
*   **No Network Starvation:** Downloading tarballs and fetching metadata is network-I/O bound (Tokio async). Decompressing tarballs (`zlib-ng`) and validating SHA-512 integrity is CPU-bound. 
*   **Strict Offloading:** You must NEVER run SHA-512 hashing or tarball decompression on the Tokio async executor thread. Always wrap heavy CPU tasks in `tokio::task::spawn_blocking` to keep the network saturated.
*   **Deterministic URLs:** Do not trust `dist.tarball` URLs from the NPM registry. Always construct tarball URLs mathematically from the package name and version to prevent cache poisoning.

### The EMFILE Shield
*   Synchronous file operations in the JS ecosystem (e.g., Vite dependency optimization) will crash the OS with `EMFILE` (Too many open files).
*   The runtime mitigates this natively. If you add new host file I/O operations, they must acquire a permit from the global `FS_SEMAPHORE` in `crates/runtime/src/io/backend.rs` to backpressure the OS.

### Hermetic by Default
*   Tests and bare `meow run` executions are mathematically deterministic. 
*   The system clock is frozen, the RNG is seeded with ChaCha20, and the OS environment is hidden.
*   **Zero-FFI Proxies:** Do not pay the FFI tax for `Math.random` or `Date.now` if the user has explicitly bypassed security (e.g., via `--trust`). Check `op_hermetic_status` once at initialization, and route to the native V8 C++ intrinsics if the cage is open.

### Fragile Runtime, Loader, and Toolchain Seams
These are not suggestions. They are scars from real regressions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xchasercat/meow](https://github.com/0xchasercat/meow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->

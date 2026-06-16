---
trigger: always_on
description: Safe Rust bindings to the [manifold3d](https://github.com/elalish/manifold) geometry kernel.
---

# manifold-csg

Safe Rust bindings to the [manifold3d](https://github.com/elalish/manifold) geometry kernel.

## Structure

- `crates/manifold-csg-sys/` — Raw C FFI bindings (`links = "manifold"`)
- `crates/manifold-csg/` — Safe Rust wrapper (the primary public API)
- `crates/manifold-csg-sys/wasm32-uu/` — Vendored helper files (config_site, mutex stub, libcxx-extras.cpp, iostream-stripping patches) used only when building for `wasm32-unknown-unknown`. See [`docs/plans/wasm-unknown-unknown.md`](docs/plans/wasm-unknown-unknown.md).
- `crates/manifold-csg-playground/` — Browser-based demo (`publish = false`); also serves as a real-world consumer test for the `wasm32-unknown-unknown` build path. cdylib that exposes a tiny C ABI to a three.js frontend (`web/`) for interactively booleaning two primitives, with Node-side unit tests under `tests/` covering the wasm ABI and the JS/three.js glue.
- `docs/plans/` — Design docs for in-flight or speculative work (e.g. new target support, large refactors). Lives in the repo so it travels with branches and stays reviewable; preferred over scattered GitHub issue prose for anything bigger than a paragraph.

## Build

The sys crate clones manifold3d (currently pinned to v3.5.1) via git and builds with cmake. Requires:
- git, cmake, a C++ compiler
- First build is slow (clones + compiles manifold3d); subsequent builds are cached

### Offline / bring-your-own manifold (issue #49)

For sandboxed builders (Nix, airgapped CI) that can't run the build script's `git clone`, an env-var escape hatch bypasses the fetch. See [`docs/plans/offline-build.md`](docs/plans/offline-build.md).

- **`MANIFOLD_CSG_LIB_DIR`** - link a pre-built manifold install (dir containing `libmanifoldc` + `libmanifold`); skips clone **and** cmake. Fully offline, no C++ compile. Optional `MANIFOLD_CSG_LIB_KIND` (`dylib` default, or `static`). Works because our FFI is hand-written `extern` decls (no bindgen), so only libraries are needed - no headers. The caller owns version/flag matching against `MANIFOLD_VERSION`. `dylib` needs the lib dir on the runtime search path (rpath); Nix `buildInputs = [ manifold ]` handles this automatically.

A source-tree override (`MANIFOLD_CSG_SOURCE_DIR`, skip clone but still cmake) is deferred - see the design doc - because our `build/build.rs` hardcodes builtin Clipper2/TBB, which `FetchContent`-clones at configure time, so it wouldn't be truly offline without a system-deps lever.

A repo-root `flake.nix` exposes a devShell that links nixpkgs' prebuilt `manifold` (3.5.1) via `MANIFOLD_CSG_LIB_DIR`; the `nix-offline` CI job (`.github/workflows/ci.yml`) runs `nix develop -c cargo test` through it, exercising the offline hatch. No buildable `packages.default` - `buildRustPackage` would need a committed `Cargo.lock` (gitignored here by convention).

## Versioning

- **`manifold-csg-sys`** uses version `{major}.{minor}.{patch}` where major.minor tracks the upstream manifold3d version and patch >= 100 is our release number. For example, `3.4.100` tracks manifold3d v3.4.1, and `3.4.101` would be our next release against the same upstream. Patch bumps (e.g., `3.4.101` → `3.4.102`) must be semver-compatible: no removed or changed function signatures, only additions.
- Exception: `3.5.101` completes `ManifoldError` to match bundled manifold3d 3.5.0 status codes (`InvalidTangents`, `Cancelled`), marks `ManifoldError` `#[non_exhaustive]`, and tightens the public `ManifoldMeshGLOptions` / `ManifoldMeshGL64Options` input pointers from `*mut` to `*const`. These are documented patch-level binding corrections: the previous incomplete enum could construct invalid Rust enum values from safe status queries, the non-exhaustive marker prevents recurring downstream match breaks when upstream adds status codes, and the option pointers are input-only fields that should not require safe wrappers to cast shared slices to mutable pointers.
- **`manifold-csg`** uses standard semver (`0.1.0`, etc.) independent of the upstream version. Its `Cargo.toml` pins the sys crate version it depends on.
- When bumping the manifold3d pin in `build.rs`, the sys crate version must be updated to match (e.g., manifold3d v3.5.0 -> sys crate `3.5.100`).
- When bumping `MANIFOLD_VERSION`, also run `nix flake update` and commit the new `flake.lock`: the `nix-offline` lane links nixpkgs' prebuilt `manifold`, which must resolve to the new pinned version or the lane links an ABI-mismatched library. The committed lock is what makes the "nixpkgs manifold matches our pin" guarantee reproducible (see [`docs/plans/offline-build.md`](docs/plans/offline-build.md)).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zmerlynn/manifold-csg](https://github.com/zmerlynn/manifold-csg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->

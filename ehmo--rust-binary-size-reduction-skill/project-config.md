---
trigger: always_on
description: Reduce Rust binary size safely across CLIs, servers, libraries, WASM targets, and embedded systems. Use when asked to shrink, strip, slim, optimize, or audit Rust build artifacts, Cargo profiles, dependency trees, monomorphization, or post-build packing.
---


# Shrink Rust Binary

Deterministic and idempotent Rust binary size reduction. Every change is
measured, reversible, and explained. No change is applied blindly.

## Principles

- **Measure first, change second.** Every optimization is bracketed by a size measurement.
- **Idempotent.** Running this skill twice produces the same result. Settings are set to exact values, not toggled.
- **Deterministic.** No randomness, no heuristics that vary between runs. Same codebase = same output.
- **Correctness over size.** Never break functionality. Flag behavioral changes explicitly.
- **Layered.** Techniques are grouped into tiers: Safe (no behavior change), Behavioral (changes panic/debug behavior), Nightly (requires nightly toolchain), and Structural (code changes).

---

## Step 0: Reconnaissance

Before changing anything, gather the full picture.

1. **Find Cargo.toml files:**
   ```bash
   find . -name Cargo.toml -not -path '*/target/*' | head -20
   ```
   Identify the workspace root vs member crates.

2. **Read the root `Cargo.toml`** and any `[profile.release]` section. Record every existing setting.

3. **Read `.cargo/config.toml`** if it exists. Check for existing RUSTFLAGS, linker settings, build-std config.

4. **Check toolchain:**
   ```bash
   rustc --version && cargo --version
   rustup show active-toolchain
   ```
   Record whether stable or nightly is active. This determines which tiers are available.

5. **Measure baseline binary size and compile time:**
   ```bash
   cargo clean --release 2>/dev/null
   time cargo build --release 2>&1 | tail -5
   ```
   Then for each binary target:
   ```bash
   ls -la target/release/<binary-name> | awk '{print $5, $9}'
   ```
   Record the **exact byte count** as the baseline and the **compile time** in seconds. LTO and codegen-units=1 significantly increase compile time (2-10x), so users need this to evaluate the trade-off.

6. **Check for existing strip/debug settings** that may already be applied. Read the full `[profile.release]` block.

7. **Check workspace member overrides:** If this is a workspace, check member crates' `Cargo.toml` files for `[profile.release]` sections that may override workspace root settings. Profile settings in member crates take precedence.

8. **Classify existing optimization level:**
   - **None:** No `[profile.release]` section or only default values → full optimization potential
   - **Partial:** Some settings present (e.g., `strip = true` but no LTO) → moderate potential
   - **Well-optimized:** Already has strip + LTO + codegen-units=1 → only opt-level and behavioral changes remain

   This classification determines which tiers will produce meaningful gains.

9. **Present the reconnaissance report:**
   ```
   ## Baseline Report
   - Toolchain: <stable/nightly version>
   - Binary: <name> = <N> bytes (<human readable>)
   - Existing profile.release settings: <list or "none">
   - Optimization level: <none/partial/well-optimized>
   - .cargo/config.toml: <exists/absent, relevant settings>
   - Workspace: <yes/no, N members>
   - Member profile overrides: <list or "none">
   ```

---

## Step 1: Safe Profile Settings (Tier 1 — No Behavior Change)

These settings affect only optimization strategy and debug metadata. They do not change runtime behavior.

### 1a. Strip debug symbols

**What:** Removes debug symbols and symbol tables from the binary. Does NOT affect runtime behavior. Cargo >= 1.59.

**Why:** Pre-compiled libstd ships with ~4 MB of DWARF debug symbols that get linked into every binary. Even with `debug = false`, libstd symbols persist unless explicitly stripped. As of Rust 1.77+, `strip = "debuginfo"` is the default for release when no debuginfo is requested anywhere, but many projects still pin older toolchains or have custom profiles.

**Setting:**
```toml
[profile.release]
strip = true
```

`strip = true` is equivalent to `strip = "symbols"` — removes both debuginfo AND symbol names. Use `strip = "debuginfo"` if you need symbol names for profiling/backtraces.

**Trade-off:** Backtraces in release builds will show only addresses, not function names or line numbers. For production binaries where panics are caught upstream, this is acceptable. For CLIs where users report panics, consider `strip = "debuginfo"` instead.

**Expected savings:** 30-90% for small binaries (libstd debuginfo dominates); 5-15% for large binaries.

After applying, rebuild and measure:
```bash
cargo build --release 2>&1 | tail -3
ls -la target/release/<binary> | awk '{print $5, $9}'
```

### 1b. Enable Link-Time Optimization (LTO)

**What:** Allows LLVM to optimize across crate boundaries at link time. Removes dead code that per-crate compilation cannot detect. Stable since Rust 1.0.

**Why:** Without LTO, each crate is optimized in isolation. Functions pulled from dependencies but never called survive in the binary. LTO sees the whole program and eliminates them.

**Setting:**
```toml
[profile.release]
lto = true
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ehmo/rust-binary-size-reduction-skill](https://github.com/ehmo/rust-binary-size-reduction-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

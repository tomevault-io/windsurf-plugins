---
trigger: always_on
description: Use when cross-arch dispatch references the function without cfg guards.
---

# archmage

> Safely invoke your intrinsic power, using the tokens granted to you by the CPU. Cast primitive magics faster than any mage alive.

## CRITICAL: Every Conversation Health Check

**Run these checks at the start of every conversation, even if the user doesn't ask:**

```bash
just generate           # Regenerate all code from token-registry.toml
just validate-registry  # Validate token-registry.toml
just validate-tokens    # Validate magetypes safety + summon() checks
just soundness          # Static intrinsic soundness verification
```

If any fail, fix them before starting other work. Report failures to the user.

## CRITICAL: Naming Conventions

**Use the thematic names, not the boring ones:**

| ❌ Don't use | ✅ Use instead | Notes |
|-------------|----------------|-------|
| `#[simd_fn]` | `#[arcane]` | `simd_fn` exists only for migration |
| `try_new()` | `summon()` | `try_new` exists only for migration |

**We are mages, not bureaucrats.** Write `Token::summon()`, not `Token::try_new()`.

### Descriptive Aliases (for AI-assisted coding)

These aliases exist so AI tools can infer behavior from the name. **Prefer the thematic names** in hand-written code, but accept both in reviews and docs.

| Thematic | Descriptive Alias | What it does |
|----------|------------------|--------------|
| `#[arcane]` | `#[token_target_features_boundary]` | Generates safe `#[target_feature]` wrapper (entry point) |
| `#[rite]` / `#[rite(v3)]` | `#[token_target_features]` | Adds `#[target_feature]` + `#[inline]` directly (internal helper). Tier-based: `#[rite(v3)]` — no token param needed |
| `incant!` | `dispatch_variant!` | Runtime dispatch to architecture-specific variants |

## Reference: CPU Features, Detection, and Dispatch

### The Core Distinction: Compile-Time vs Runtime

| Mechanism | When | Effect |
|-----------|------|--------|
| `#[cfg(target_arch = "...")]` | Compile | Include/exclude code from binary |
| `#[cfg(target_feature = "...")]` | Compile | True only if feature is in target spec |
| `#[cfg(feature = "...")]` | Compile | Cargo feature flag |
| `-Ctarget-cpu=native` | Compile | LLVM assumes current CPU's features |
| `is_x86_feature_detected!()` | Runtime | CPUID instruction |
| `Token::summon()` | Runtime | Archmage's detection (compiles away when guaranteed) |

**Tokens exist everywhere.** `X64V3Token`, `Arm64`, etc. compile on all platforms—`summon()` just returns `None` on unsupported architectures. `#[arcane]`/`#[rite]` cfg-gate their output to the matching architecture automatically, so you don't need `#[cfg(target_arch)]` on function definitions. `incant!` also handles cfg-gating at call sites.

### CRITICAL: How the Macros Choose Features

`#[arcane]` and `#[rite]` determine features in three ways:

1. **Token-based** (default): Parse the token type from the function signature. `X64V3Token` → `#[target_feature(enable = "avx2,fma,...")]`.
2. **Tier-based** (`#[rite(v3)]`): The tier name specifies the features directly. No token parameter needed.
3. **Multi-tier** (`#[rite(v3, v4, neon)]`): Generates a suffixed variant for each tier (`fn_v3`, `fn_v4`, `fn_neon`), each with its own `#[target_feature]` and `#[cfg(target_arch)]`.

Single-tier and token-based produce identical `#[target_feature]` attributes. Multi-tier produces multiple functions — one per tier, each compiled with different features. The token form can be easier to remember if you already have the token in scope.

`#[arcane]` generates a wrapper: an outer function that calls an inner `#[target_feature]` function via `unsafe`. This wrapper is how you cross into SIMD code without writing `unsafe` yourself — but it also creates an LLVM optimization boundary. `#[rite]` applies `#[target_feature]` + `#[inline]` directly, with no wrapper and no boundary.

**`#[rite]` should be the default.** Use `#[arcane(import_intrinsics)]` only at the entry point. For internal helpers, use `#[rite(v3, import_intrinsics)]` (tier-based, no token parameter) or `#[rite(import_intrinsics)]` (token-based). For multi-tier auto-vectorization, use `#[rite(v3, v4, neon)]`. `import_intrinsics` auto-imports `archmage::intrinsics::{arch}::*` — a combined module where `safe_unaligned_simd` memory ops shadow `core::arch` pointer-based ones. No ambiguity, no qualification needed.

Multi-tier variants are safe to call from matching `#[arcane]` or `#[rite]` contexts — since Rust 1.86, `#[target_feature]` functions can safely call other `#[target_feature]` functions when the caller has matching or superset features.

### CRITICAL: Target-Feature Boundaries (4x Performance Impact)

**Enter `#[arcane(import_intrinsics)]` once at the top, use `#[rite(import_intrinsics)]` for everything inside.**

LLVM cannot inline across mismatched `#[target_feature]` attributes. Each `#[arcane]` call from non-SIMD code creates an optimization boundary — LLVM can't hoist loads, sink stores, or vectorize across it. This costs 4-6x depending on workload (see `benches/asm_inspection.rs` and `docs/PERFORMANCE.md`). Token hoisting doesn't help — even with the token pre-summoned, calling `#[arcane]` per iteration still hits the boundary.

```rust
// WRONG: #[arcane] boundary every iteration (4x slower)
#[arcane(import_intrinsics)]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imazen/archmage](https://github.com/imazen/archmage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

---
trigger: always_on
description: **Quality through iterations** - Start with correct implementations, then improve. Code that doesn't matter can be quick and dirty. But code that matters *really* matters—treat it with respect and improve it meticulously. In a porting project, every function matters.
---

# Claude Code Guidelines

## Philosophy

**Quality through iterations** - Start with correct implementations, then improve. Code that doesn't matter can be quick and dirty. But code that matters *really* matters—treat it with respect and improve it meticulously. In a porting project, every function matters.

**Circumstances alter cases** - Use judgment. There are no rigid rules—context determines the right approach. However, this project has strong defaults because porting demands precision.

**No stubs, no shortcuts** - Every function must be complete and production-ready. No `todo!()`, no `unimplemented!()`, no `panic!("not implemented")`, no partial implementations. If dependencies aren't ready, stop and implement them first.

## Test-First Bug Fixing (Critical Practice)

**This is the single most important practice for agent performance and reliability.**

When a bug is reported, always follow this workflow:

1. **Write a reproducing test first** - Create a test that fails, demonstrating the bug
2. **Fix the bug** - Make the minimal change needed to address the issue
3. **Verify via passing test** - The previously failing test should now pass

**Do not skip the reproducing test.** Even if the fix seems obvious, the test validates your understanding and prevents regressions.

## Testing

- Tests MUST test actual production code, not copies - Never duplicate production logic in tests. Import and call the real code.
- Tests should run as fast as possible—fast tests get run more often
- Write tests for regressions and complex logic
- All tests must pass before merging
- Tests must verify **exact behavioral match** with the C implementation
- Port the C test suite (`box3d-cpp-reference/test/`) module by module alongside the code it tests
- When test failures occur, use the fix-test-failures agent (`.claude/agents/fix-test-failures.md`) — it treats all failures as real bugs and resolves them through instrumentation and root cause analysis, never by weakening tests

**Running tests:**
```bash
# Run with the double-precision (large world) feature, mirroring BOX3D_DOUBLE_PRECISION
cargo test --features double-precision
```

## Code Quality

**Names** - Follow Rust conventions (`snake_case` for functions/variables, `PascalCase` for types, `SCREAMING_SNAKE_CASE` for constants). Keep names mappable to the C original: `b3ClampFloat` → `clamp_float`, `b3Vec3` → `Vec3`, `b3InvMulTransforms` → `inv_mul_transforms`. A reader diffing Rust against C should never have to guess which function corresponds to which.

**Comments** - Preserve the C source's explanatory comments (they encode Erin Catto's reasoning). Comments explaining *why* the Rust approach differs from C are especially valuable.

**Refactoring** - A tripped file-length test means split the file into real modules — never compact lines or bump the limit.

## C to Rust Porting Rules

This project is a strict port of the Box3D C library to Rust. These rules ensure fidelity:

### The C Reference

The exact source being ported is the git submodule at `box3d-cpp-reference/` (pinned, currently v0.1.0+ at `3fc20f5`). **Always read the pinned submodule, not the upstream website** — upstream moves fast (Box3D was released June 2026). Layout:

- `src/*.c` + `src/*.h` — internal implementation (the bulk of the port)
- `include/box3d/*.h` — public API and inline functions (port these with the module that owns them)
- `test/test_*.c` — the C test suite (port alongside each module)
- `samples/` — the interactive samples app (becomes the wasm demo site)
- `benchmark/` — performance tests (informs the Rust benches later)

Box3D is a 3D fork of Box2D v3. When a Box3D function is unclear, the corresponding Box2D
function (and our finished box2d-rust port at `../box2d-rust/`) is often the best Rosetta
stone — but the pinned Box3D source is always the authority.

### Exact Behavioral Matching
- Rust implementation must match C behavior exactly: same algorithms, same arithmetic, same edge cases
- **Float precision is behavior.** Box3D does all math in `f32`. Never promote to `f64` for "extra accuracy", never reorder floating-point operations, never replace `x * x + y * y` with `hypot`. `(a + b) + c ≠ a + (b + c)` in floats.
- **Determinism is a feature.** Box3D hand-rolls `b3Atan2` and `b3ComputeCosSin` specifically for cross-platform determinism. Never replace them with `f32::atan2` / `f32::sin_cos` — port the approximations bit-for-bit.
- **Port the scalar path.** Upstream has SIMD (SSE2/Neon) in `src/simd.h` with a scalar fallback selected by `BOX3D_DISABLE_SIMD` / `B3_SIMD_NONE`. The port implements the scalar (`B3_SIMD_NONE`) path — it is the behavioral reference. SIMD acceleration may come later only if it stays bit-identical to the scalar results.
- **The port is serial.** Upstream's task scheduler (`scheduler.c`, `parallel_for.c`) runs worker threads; the port executes the same work single-threaded in the same order, like box2d-rust does.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [larsbrubaker/box3d-rust](https://github.com/larsbrubaker/box3d-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->

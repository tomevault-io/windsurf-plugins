---
trigger: always_on
description: transforms, physics, BVH traversal, culling, animation, etc.
---

# vecmath - SIMD Math Library

## Overview
Platform-abstracted SIMD vector math library. Wraps SSE2/SSSE3/SSE4.1 (x86), NEON (ARM) and a
scalar per-lane fallback for targets with no SIMD ISA behind a
unified C API. Used pervasively throughout the Dagor Engine for all performance-critical math:
transforms, physics, BVH traversal, culling, animation, etc.

## Key Types (dag_vecMathDecl.h)
- `vec4f` / `vec3f` -- 128-bit float vector (__m128 on SSE, float32x4_t on NEON, a 16-byte struct on scalar)
- `vec4i` -- 128-bit integer vector (__m128i / int32x4_t)
- `mat33f` -- 3x3 column-major matrix (3 x vec3f)
- `mat44f` -- 4x4 column-major matrix (4 x vec4f)
- `mat43f` -- 4x3 row-major matrix (3 x vec4f, each row is xyzw where w = translation component)
- `bbox3f` -- AABB {bmin, bmax} as two vec4f
- `bsph3f` -- bounding sphere {center.xyz, radius.w}
- `quat4f` -- quaternion as vec4f
- `plane3f` -- plane {normal.xyz, D.w}

## File Layout
| File | Contents |
|------|----------|
| `dag_vecMathDecl.h` | Type declarations, platform detection, alignment macros |
| `dag_vecMath.h` | Full API declarations (~650 functions) with comments, platform-independent |
| `dag_vecMath_const.h` | Constants: V_C_HALF, V_C_ONE, V_C_PI, V_C_UNIT_1000, V_CI_MASK*, etc. |
| `dag_vecMath_pc_sse.h` | SSE low-level implementation of basic functions |
| `dag_vecMath_neon.h` | NEON (ARM) low-level implementation of basic functions |
| `dag_vecMath_scalar.h` | Scalar per-lane implementation of basic functions (no-SIMD fallback, forceable with `_TARGET_SIMD_SCALAR=1`) |
| `dag_vecMath_double.h` | `vec4d` double-precision math (SSE/AVX, NEON and scalar in one file); include via dag_vecMath.h |
| `dag_vecMath_common.h` | Shared implementations (bbox, frustum, quat, matrix ops built on core intrinsics) without hw-specific intrinsics |
| `dag_vecMath_trig.h` | Polynomial approximations for sin/cos/tan/atan/asin/acos |

## API Conventions
dag_vecMath.h is the API reference: every `v_`-prefixed function is declared there with a
comment. Grep it by prefix before writing a new helper - what you need probably exists
(v_mat44_*, v_mat33_*, v_mat43_*, v_quat_*, v_bbox3_*, v_bsph*, v_plane*, v_frustum*, v_ray*,
v_triangle*).

- Suffix scheme:
  - `_x` -- result in .x only, .yzw are allowed to contain anything (even NaN); cheaper than the
    broadcast form
  - `_est` -- hardware estimate refined by Newton-Raphson steps; sequence and step count vary
    per op and platform to land at similar useful precision (not an automatic win, see microarch
    notes)
  - `_unprecise` -- raw hardware estimate only (fastest, least precise)
  - `_safe` -- tolerates edge inputs (zero divisors, page-boundary loads); the divisor check
    (v_is_unsafe_divisor: |x| < V_C_VERY_SMALL_VAL, 4e-19 ~ sqrt(FLT_MIN)) is tuned to stay safe
    under squaring: if x passes, x*x does not underflow and 1/(x*x) does not overflow
- Basic rcp/sqrt/rsqrt use real division/square-root instructions, never hardware estimate
  instructions; estimates enter only through the _est/_unprecise variants
- 3-component ops ignore .w of inputs: it can be anything, even NaN. Most gameplay code safely
  carries 3d vectors in vec3f with garbage in .w, but some render code requires .w = 0. Same for
  mat44f holding a 4x3 transform: row3 (the .w lane of the columns) can be anything for gameplay,
  but the GPU requires 0,0,0,1 there
- Loads/stores: v_ld/v_st aligned, v_ldu/v_stu unaligned, *i for integer. On modern cores
  unaligned is as fast as aligned within a cache line, so aligned forms act mostly as a hardware
  alignment assert. Encodings differ: legacy (non-VEX) SSE load+op memory operands (addps xmm,
  [mem]) require alignment, so on the SSE4.1 PC build only v_ld folds into the consuming
  instruction (v_ldu stays a separate movups); VEX operands don't require alignment, both forms
  fold, and a folded v_ld loses the movaps fault; NEON never checks alignment
- v_ldu_p3 reads 4 floats for a 3-component vector (fast): harmless on the stack, but a heap
  array of packed 3-float positions whose last element ends exactly at a page end crashes on the
  final load - use v_ldu_p3_safe (reads exactly 3, slower) there. ASAN/TSAN builds route v_ldu_p3
  to the safe path automatically, so sanitizers neither catch nor reproduce the over-read
- Store ports are scarcer than load ports (big cores sustain 2-3 loads but only 1-2 stores per
  cycle), and v_stu_p3 issues two store uops (8-byte + 4-byte); when the layout allows
  overwriting the padding, prefer one full v_st/v_stu
- Lane ops: v_splats(float) / v_splatsi(int) broadcast a scalar, v_splat_x/y/z/w broadcast a lane,
  v_extract_x/y/z/w read a lane out
- Swizzles: v_perm_<lanes> with xyzw lane names; two-vector perms name the second operand's lanes
  abcd (v_perm_xzac); v_rot_1/2/3 rotate lanes. These are hand-mapped to cheap forms on both ISAs -
  prefer them over raw intrinsics or exotic custom shuffles
- Compares (v_cmp_gt/ge/lt/le/eq) return per-lane masks (all-ones or 0); consume masks with bitwise
  ops (v_and/v_or/v_xor/v_andnot), branchless selects (v_sel/v_sel_b/v_btsel/v_bbox3_sel), the
  v_check_*/v_test_* branching helpers, or v_signmask (sign bits as int)
- v_cast_* reinterprets bits (free), v_cvt_* converts values (int<->float)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaijinEntertainment/vecmath](https://github.com/GaijinEntertainment/vecmath) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

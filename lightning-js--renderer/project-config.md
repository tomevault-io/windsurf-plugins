---
trigger: always_on
description: **Target**: High-performance JavaScript rendering engine for constrained embedded browser environments
---

# Lightning 3 Renderer - Agent Development Instructions

**Target**: High-performance JavaScript rendering engine for constrained embedded browser environments

## Core Philosophy

**Performance > Readability**: We optimize for raw performance. Readability is secondary.

### Architecture Principles

- **Class-based design** - Use TypeScript classes for structure and type safety
- **Zero GC pressure** - Minimize object allocation, reuse everything possible
- **Direct memory management** - Pre-allocate buffers, use typed arrays
- **Zero safety checks** - Input validation is caller's responsibility
- **Early returns** - Most common paths first, error checks on top
- **Arrow functions for utilities** - Use arrow functions for libraries/singletons/utilities

## GPU Target: Mali 400 / OpenGL ES 2.0 Class Hardware

This renderer targets **Mali 400-class GPUs** (and comparable OpenGL ES 2.0-class silicon such as Vivante GC400, PowerVR SGX). These are in-order, tile-based, scalar fragment pipelines with strict GLSL ES 1.0 constraints. Every shader decision must be evaluated against this baseline. Modern GPUs are a bonus, not the target.

### WebGL / GLSL Hard Constraints

- **WebGL 1.0 only** — no WebGL 2.0 APIs (`textureLod`, `textureOffset`, integer attributes, MRT, `gl.UNSIGNED_INT` indices, etc.)
- **Fragment shader precision** — Mali 400 has no `highp` in fragment shaders. Always use the `#ifdef GL_FRAGMENT_PRECISION_HIGH` guard that is already the project standard. Never assume `highp` is available in a fragment shader.
- **Index buffers** — always use `Uint16Array` / `gl.UNSIGNED_SHORT`. `gl.UNSIGNED_INT` requires `OES_element_index_uint` which is absent on Mali 400.
- **Max 8 texture units** — keep sampler count at or below 8. Do not index sampler arrays with a non-constant (varying or computed) expression — this is undefined behavior in GLSL ES 1.0 and silently breaks on Mali 400 drivers.
- **Max 8 vec4-equivalent varyings** — count varyings before adding new ones. Each `vec4` or `vec2 + vec2` pair counts as one unit toward the limit of 8. Exceeding this will silently drop varyings or fail to compile on constrained drivers.
- **NPOT textures** — non-power-of-two textures must use `CLAMP_TO_EDGE` wrapping and must not use mipmaps.
- **No derivative instructions** — `dFdx`, `dFdy`, and `fwidth` require `OES_standard_derivatives`. Check for the extension before use; do not use in core shaders.

### Avoid Branching in GLSL Shaders (CRITICAL)

Mali 400 has an in-order scalar fragment pipeline. Any `if`, `else`, or ternary `?:` in a fragment shader causes the pipeline to serialize both code paths for every fragment in the affected batch. This is not a style concern — it is a frame-time concern.

**Rule: never use `if`, `else`, or `?:` in a fragment shader.** Use arithmetic equivalents instead.

```glsl
// ❌ AVOID: if/else branch in fragment shader
if (v_texcoord.x < 0.0) {
  gl_FragColor = solidColor;
} else {
  gl_FragColor = texture2D(u_texture, v_texcoord);
}

// ✅ DO: arithmetic mask — no branch, both paths are cheap
float isSolid = step(0.0, -v_texcoord.x);   // 1.0 when x < 0, else 0.0
gl_FragColor = mix(texture2D(u_texture, v_texcoord), solidColor, isSolid);
```

```glsl
// ❌ AVOID: ternary in fragment shader (it is a branch in GLSL ES 1.0)
r.xy = (p.x > 0.0) ? r.yz : r.xw;

// ✅ DO: mix + step
float s = step(0.0, p.x);
r.xy = mix(r.xw, r.yz, s);
```

```glsl
// ❌ AVOID: early return from fragment main() or fragment functions
float getGradientColor(float dist) {
  if (dist <= u_stops[0]) return u_colors[0];  // early exit — branch
  for (int i = 0; i < LAST_STOP; i++) {
    if (dist >= u_stops[i] && dist <= u_stops[i+1]) {
      return mix(u_colors[i], u_colors[i+1], ...); // return inside loop — UB risk
    }
  }
  // missing final return — undefined behavior in GLSL ES 1.0
}

// ✅ DO: branchless accumulation — single return, no conditionals
vec4 getGradientColor(float dist) {
  vec4 color = u_colors[0];
  for (int i = 0; i < LAST_STOP; i++) {
    float t = clamp((dist - u_stops[i]) / (u_stops[i+1] - u_stops[i]), 0.0, 1.0);
    color = mix(color, mix(u_colors[i], u_colors[i+1], t), step(u_stops[i], dist));
  }
  return color;
}
```

Additional rules:

- **No `discard`** in fragment shaders — kills early-Z optimizations and stalls the tile pipeline. Use `mix()` with a zero-alpha result instead.
- **No `for` loops with non-compile-time-constant bounds** in shaders — loop counts must be a literal or `#define` constant resolvable at compile time.
- **No sampler array indexing by varying** — `texture2D(u_textures[int(v_index)], uv)` is undefined behavior in GLSL ES 1.0. Use a compile-time-unrolled `if/else` chain per texture unit, or restructure to avoid batched multi-texture lookups entirely.
- **All non-void GLSL functions must have a `return` on every code path** — a function that can exit without returning is undefined behavior in GLSL ES 1.0 and will produce incorrect output on Mali 400.

### CPU-side WebGL for Tile-Based Renderers

Tile-based GPUs (Mali, PowerVR, Adreno low-end) pay a heavy cost for certain API patterns that are cheap on desktop GPUs:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lightning-js/renderer](https://github.com/lightning-js/renderer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

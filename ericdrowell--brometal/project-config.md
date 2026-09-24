---
trigger: always_on
description: You are writing an app that uses BroMetal. This file is the whole orientation:
---

# BroMetal — notes for coding agents

You are writing an app that uses BroMetal. This file is the whole orientation:
what it is, the rules the DSL enforces, and the mistakes that fail *silently*.

BroMetal compiles shaders written in a typed TypeScript DSL into WGSL **at build
time**, and ships a WebGPU runtime behind one API. There is no scene graph, no
material system, and no shader compiler in the browser. You write the shader.

---

## The loop

1. Write `src/shaders/thing.shader.ts` — `export const Thing = shader({...})`.
   Name it explicitly; `--js13k` requires the name and uses it as the global.
2. Run `npx brometal dev` (watch) or `npx brometal prod` (one-shot, optimized).
   This writes `thing.shader.gen.ts` next to it.
3. Import the **`.gen`** module in your app, never the `.shader.ts` source.

**If you edit a `.shader.ts` you must recompile before the change has any
effect.** Nothing at runtime reads the source. A common failure is editing a
shader, reloading, and concluding the change did nothing.

```ts
import { createRenderer, createProgram, createCamera, createCube } from 'brometal';
import cubeShader from './shaders/cube.shader.gen';

const renderer = await createRenderer(canvas);      // WebGPU; throws where unavailable
const program = createProgram(renderer, cubeShader);
const cube = createCube({ width: 1, height: 1, depth: 1 });

program.attributes.aPosition.set(cube.positions);
program.setIndices(cube.indices);

renderer.loop((t) => {
  program.uniforms.uViewProj.set(camera.viewProjection(renderer.aspect));
  program.draw();
});
```

---

## Rules the DSL enforces

These are compile errors with `file:line:col`. Read the message; it names the fix.

- **One `return`**, as the final top-level statement of `vertex` / `fragment`.
  No early returns, no returns inside `if`.
- `vertex` returns a `vec4` clip position. `fragment` returns a `vec4` colour.
- `vertex` must assign **every** declared varying.
- **No arrays, no structs.** Loop over a texture instead.
- `for` loops need a float counter: `for (let i = 0; i < n; i += 1)`.
- `if` / `else if` / `else` are fine. No `while`, no `switch`, no `break`.
- Uniform types: `float vec2 vec3 vec4 mat4 sampler2D`. Attributes and varyings
  cannot be `mat4` or `sampler2D`.
- Helper functions must be **module-level** `function` declarations with typed
  params, declared **above first use**. Params may be
  `number, Vec2, Vec3, Vec4, Mat4, Sampler2D`.

### Float-only vs vector intrinsics

This one bites constantly. These take **float arguments only** — passing a
vector is a compile error:

```
sin  cos  tan  asin  acos  atan  abs  sign  fract  floor  sqrt  exp  exp2  log
pow  min  max  mod  step  smoothstep
```

These take vectors and **return a float**: `length(v)`, `dot(a, b)`,
`distance(a, b)`.

These take vectors and return vectors: `normalize`, `cross`, `reflect`,
`texture`, `targetUv`, and the `vec2/vec3/vec4` constructors. `mix(a, b, t)`
takes float-or-vector `a`/`b` with a float `t`; `clamp(x, min, max)` takes a
float-or-vector `x` with float bounds.

For a component-wise `max` on a vector, do it per component:
`vec3(max(v.x, 0), max(v.y, 0), max(v.z, 0))`.

### Vector maths is method calls, not operators

`a.add(b) a.sub(b) a.mul(b) a.div(b) a.scale(k)`, and `m.mul(v)` for a `mat4`.
`a + b` on two vectors will not compile. Swizzles read normally: `v.xyz`, `v.x`.

### Unary minus on a vector

Write `0 - x` rather than `-x` when negating an expression you are unsure about;
`vec3(0,1,0).scale(0 - 1)` is the reliable form.

---

## Failures that are silent — read this before debugging a black screen

**Reserved words.** WGSL reserves a long list of ordinary-looking words for
future use, and several are names you would reach for without thinking:
`type set get from new use with where match self null pass target filter
precise shared mod`. BroMetal rejects them at build time, so this shows up as a
name error rather than a black screen — but that is why.

**`texture()` cannot be called inside an `if`.** WGSL requires texture sampling
to happen in uniform control flow. Sampling inside a conditional invalidates the
*whole pipeline* — the pass silently draws nothing, and you get a black screen
with no console error. Sample unconditionally and multiply the result away:

```ts
const contribution = texture(uMap, uv).x * step(0.5, someCondition);
```

(Inside a **helper function** the emitter uses `textureSampleLevel`, which does
not carry that restriction — but it always samples LOD 0, so no mipmapping.)

**Sampling a render target needs `targetUv`.** NDC +y lands on a target's *first*
row while texture v runs top-down, so hand-rolling `clip.xy / clip.w * 0.5 + 0.5`
reads the target **vertically mirrored**. A mirrored lookup still produces a
plausible-looking image, so this reads as a lighting bug rather than a coordinate
one.

```ts
const uv = targetUv(lightClipPosition);
```

**The canvas has no `width`/`height` attributes.** BroMetal owns the drawing
buffer and tracks the CSS box at the device pixel ratio. Size it with CSS, and
give the *container* a definite size. Setting the attributes plants an intrinsic
minimum size that overflows flex containers.

---

## Shadows


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ericdrowell/brometal](https://github.com/ericdrowell/brometal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

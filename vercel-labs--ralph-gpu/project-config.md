---
trigger: always_on
description: Complete guide for using ralph-gpu - a minimal WebGPU shader library for creative coding and real-time graphics.
---


# ralph-gpu Usage Guide

ralph-gpu is a WebGPU shader library for creative coding and real-time graphics. This rule provides comprehensive guidance on using the library effectively.

## Installation

```bash
npm install ralph-gpu
# or
pnpm add ralph-gpu
```

For TypeScript support (recommended):

```bash
npm install -D @webgpu/types
```

## Core Concepts

| Concept    | Description                                             |
| ---------- | ------------------------------------------------------- |
| `gpu`      | Module entry point for initialization                   |
| `ctx`      | GPU context — manages state and rendering               |
| `pass`     | Fullscreen shader (fragment only, uses internal quad)   |
| `material` | Shader with custom vertex code (particles, geometry)    |
| `target`   | Render target (offscreen texture)                       |
| `pingPong` | Pair of render targets for iterative effects            |
| `compute`  | Compute shader for GPU-parallel computation             |
| `storage`  | Storage buffer for large data (particles, simulations)  |
| `sampler`  | Custom texture sampler with explicit filtering/wrapping |
| `texture`  | Load images, canvases, video, or raw data as GPU textures |

## Auto-Injected Globals

Every shader automatically has access to these uniforms via `globals`:

```wgsl
struct Globals {
  resolution: vec2f,  // Current render target size in pixels
  time: f32,          // Seconds since init (affected by timeScale)
  deltaTime: f32,     // Seconds since last frame
  frame: u32,         // Frame count since init
  aspect: f32,        // resolution.x / resolution.y
}
@group(0) @binding(0) var<uniform> globals: Globals;
```

**Usage in shaders:**

```wgsl
let uv = pos.xy / globals.resolution;        // Normalized UV coordinates
let t = globals.time;                         // Animated time
let dt = globals.deltaTime;                   // Frame delta
let ar = globals.aspect;                      // Aspect ratio
```

## Basic Patterns

### 1. Initialization with React

```tsx
"use client";

import { useEffect, useRef } from "react";
import { gpu, GPUContext, Pass, Sampler, RenderTarget } from "ralph-gpu";

export default function ShaderComponent() {
  const canvasRef = useRef<HTMLCanvasElement>(null);

  useEffect(() => {
    let ctx: GPUContext | null = null;
    let pass: Pass;
    let animationId: number;
    let disposed = false;

    async function init() {
      if (!canvasRef.current) return;

      // Always check WebGPU support first
      if (!gpu.isSupported()) {
        console.error("WebGPU is not supported");
        return;
      }

      ctx = await gpu.init(canvasRef.current, {
        autoResize: true, // Automatically handles canvas sizing and DPR
        debug: true,
      });

      // Handle disposal during async init
      if (disposed) {
        ctx.dispose();
        return;
      }

      pass = ctx.pass(/* wgsl */ `
        @fragment
        fn main(@builtin(position) pos: vec4f) -> @location(0) vec4f {
          let uv = pos.xy / globals.resolution;
          return vec4f(uv, sin(globals.time) * 0.5 + 0.5, 1.0);
        }
      `);

      function frame() {
        if (disposed) return;
        pass.draw();
        animationId = requestAnimationFrame(frame);
      }
      frame();
    }

    init();

    return () => {
      disposed = true;
      cancelAnimationFrame(animationId);
      ctx?.dispose();
    };
  }, []);

  return (
    <canvas
      ref={canvasRef}
      style={{ width: "100%", height: "100%" }}
      width={800}
      height={600}
    />
  );
}
```

### 2. Fullscreen Pass (Fragment Shader)

Use `ctx.pass()` for fullscreen effects. It has two modes:

#### Simple Mode (Recommended)

Pass an object with plain values. WGSL bindings are auto-generated and uniforms are available via the `uniforms` struct.

```tsx
const wave = ctx.pass(
  /* wgsl */ `
  @fragment
  fn main(@builtin(position) pos: vec4f) -> @location(0) vec4f {
    let uv = pos.xy / globals.resolution;
    // Bindings for uTexture, uSampler, and uniforms struct are auto-generated!
    let tex = textureSample(uTexture, uSampler, uv);
    return tex * vec4f(uniforms.color, 1.0) * uniforms.intensity;
  }
`,
  {
    uTexture: someTarget,
    color: [1, 0, 0],
    intensity: 0.5,
  }
);

// Update values directly
wave.set("intensity", 0.8);
```

#### Manual Mode (Explicit Bindings)

Define uniforms with `{ value: X }` wrapper for reactive updates. Requires manual `@group(1)` declarations in WGSL.

```tsx
const gradient = ctx.pass(
  /* wgsl */ `
  struct MyUniforms { color: vec3f }
  @group(1) @binding(0) var<uniform> u: MyUniforms;

  @fragment
  fn main(@builtin(position) pos: vec4f) -> @location(0) vec4f {
    let uv = pos.xy / globals.resolution;
    return vec4f(u.color, 1.0);
  }
`,
  {
    uniforms: {
      color: { value: [1, 0, 0] },
    },
  }
);

// Update value
gradient.uniforms.color.value = [0, 1, 0];
```

// In render loop
wave.draw();

````

### 3. Custom Uniforms (Reactive Pattern)

Define uniforms with `{ value: X }` wrapper for reactive updates:

```tsx
// Define uniforms object
const uniforms = {
  amplitude: { value: 0.5 },
  frequency: { value: 10.0 },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/ralph-gpu](https://github.com/vercel-labs/ralph-gpu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

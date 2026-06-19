---
trigger: always_on
description: >
---


# Procedural Clouds

Generate visually stunning procedural clouds in Three.js with artistic emphasis —
volumetric raymarching on WebGPU, billboard/mesh fallbacks on WebGL2.

## Architecture Overview

```
┌──────────────────────────────────────────────────────┐
│                  Cloud Pipeline                       │
│                                                      │
│  Rendering Paths (select by capability + budget):    │
│                                                      │
│  ┌─ VOLUMETRIC (WebGPU) ─────────────────────────┐   │
│  │  Fullscreen quad → raymarching fragment shader │   │
│  │  Noise: 3D worley/perlin compute textures     │   │
│  │  Best quality, most expensive                  │   │
│  └───────────────────────────────────────────────┘   │
│                                                      │
│  ┌─ MESH CLUSTER (WebGL2/WebGPU) ────────────────┐   │
│  │  Instanced soft-particle spheres              │   │
│  │  Per-instance density, color, fade            │   │
│  │  Good quality, moderate cost                   │   │
│  └───────────────────────────────────────────────┘   │
│                                                      │
│  ┌─ BILLBOARD (WebGL2, mobile) ──────────────────┐   │
│  │  Camera-facing quads with noise texture       │   │
│  │  Cheapest, suitable for backgrounds           │   │
│  └───────────────────────────────────────────────┘   │
│                                                      │
│  Shared Systems:                                     │
│  Lighting ─ Drift ─ Time-of-Day ─ Formation         │
└──────────────────────────────────────────────────────┘
```

## Cloud Classification Quick Reference

| Genus | Altitude | Shape | Key Visual |
|-------|----------|-------|------------|
| Cumulus | Low (2km) | Puffy mounds | Flat base, cauliflower tops |
| Stratus | Low (2km) | Flat sheet | Uniform grey blanket |
| Stratocumulus | Low (2km) | Lumpy rolls | Patchy blanket with gaps |
| Cumulonimbus | Low→High | Towering anvil | Massive vertical, dark base |
| Altocumulus | Mid (2-6km) | Rippled patches | "Mackerel sky" pattern |
| Altostratus | Mid (2-6km) | Thin veil | Sun visible as bright spot |
| Nimbostratus | Mid (2-6km) | Thick dark sheet | Continuous rain cloud |
| Cirrus | High (6-12km) | Wispy streaks | Ice crystal hooks and mares' tails |
| Cirrostratus | High (6-12km) | Thin milky haze | Halo around sun |
| Cirrocumulus | High (6-12km) | Tiny ripples | Delicate fish-scale pattern |

Full profiles with shader parameters in `references/cloud-types.md`.

## Renderer Setup

```javascript
import * as THREE from 'three';

async function createRenderer(canvas) {
  let renderer, gpuAvailable = false;
  try {
    const WebGPU = (await import('three/addons/capabilities/WebGPU.js')).default;
    if (WebGPU.isAvailable()) {
      const { default: WebGPURenderer } = await import(
        'three/addons/renderers/webgpu/WebGPURenderer.js'
      );
      renderer = new WebGPURenderer({ canvas, antialias: true });
      await renderer.init();
      gpuAvailable = true;
    }
  } catch (e) { /* fallback */ }
  if (!renderer) {
    renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    renderer.toneMappingExposure = 1.2;
  }
  renderer.setSize(innerWidth, innerHeight);
  renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
  return { renderer, gpuAvailable };
}
```

## 3D Noise Foundation

All cloud rendering depends on layered 3D noise. These functions are shared across
all three rendering paths.

```javascript
// GPU-friendly 3D hash (no lookup tables)
// Used in shaders — JavaScript equivalent for CPU cloud mesh placement
function hash3(x, y, z) {
  let h = x * 127.1 + y * 311.7 + z * 74.7;
  return (Math.sin(h) * 43758.5453) % 1;
}

// 3D value noise
function noise3D(x, y, z) {
  const ix = Math.floor(x), iy = Math.floor(y), iz = Math.floor(z);
  const fx = x - ix, fy = y - iy, fz = z - iz;
  const ux = fx * fx * (3 - 2 * fx);
  const uy = fy * fy * (3 - 2 * fy);
  const uz = fz * fz * (3 - 2 * fz);

  const h = (a, b, c) => hash3(ix + a, iy + b, iz + c);
  return lerp(uz,
    lerp(uy, lerp(ux, h(0,0,0), h(1,0,0)), lerp(ux, h(0,1,0), h(1,1,0))),
    lerp(uy, lerp(ux, h(0,0,1), h(1,0,1)), lerp(ux, h(0,1,1), h(1,1,1)))
  );
}
function lerp(t, a, b) { return a + t * (b - a); }

// FBM for cloud density
function cloudFBM(x, y, z, octaves = 5, lac = 2.0, gain = 0.5) {
  let sum = 0, amp = 1, freq = 1, max = 0;
  for (let i = 0; i < octaves; i++) {
    sum += noise3D(x * freq, y * freq, z * freq) * amp;
    max += amp; amp *= gain; freq *= lac;
  }
  return sum / max;
}
```

## Path 1: Volumetric Raymarching (WebGPU)

The highest-quality path renders clouds by marching rays through a density field defined
by 3D noise. Implemented as a fullscreen post-process pass.

### Cloud Density Field

The density function defines cloud shape, coverage, and type:

```javascript
// GLSL-style pseudocode for the density function (full GLSL in references)
float cloudDensity(vec3 p, float time) {
  // Altitude shaping — confine to cloud layer
  float altFade = smoothstep(cloudBase, cloudBase + 200.0, p.y)
                * smoothstep(cloudTop, cloudTop - 200.0, p.y);

  // Large-scale shape (coverage map)
  float shape = fbm3D(p * 0.0003 + wind * time, 3);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CK42BB/procedural-clouds-threejs](https://github.com/CK42BB/procedural-clouds-threejs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

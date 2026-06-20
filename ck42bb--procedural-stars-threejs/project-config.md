---
trigger: always_on
description: >
---


# Procedural Starfield & Celestial Phenomena

Generate breathtaking night skies in Three.js — from photorealistic starfields to
dreamy nebulae to dramatic celestial events.

## Architecture Overview

```
┌──────────────────────────────────────────────────────┐
│               Night Sky Pipeline                      │
│                                                      │
│  SkyController (master orchestrator)                 │
│    ├── time progression (sunset → night → dawn)      │
│    ├── moon phase + position                         │
│    └── drives all layers:                            │
│                                                      │
│  ┌─ Layer 1: Sky Dome ──────────────────────────┐    │
│  │  Gradient background, horizon glow, zodiacal │    │
│  └──────────────────────────────────────────────┘    │
│  ┌─ Layer 2: Stars ─────────────────────────────┐    │
│  │  Points with spectral color + magnitude      │    │
│  │  Twinkle, proper motion (optional)           │    │
│  └──────────────────────────────────────────────┘    │
│  ┌─ Layer 3: Milky Way ─────────────────────────┐    │
│  │  Textured band or procedural FBM glow        │    │
│  └──────────────────────────────────────────────┘    │
│  ┌─ Layer 4: Nebulae ───────────────────────────┐    │
│  │  Volumetric raymarched or billboard sprites  │    │
│  └──────────────────────────────────────────────┘    │
│  ┌─ Layer 5: Celestial Bodies ──────────────────┐    │
│  │  Moon (phase lit), planets, sun glow         │    │
│  └──────────────────────────────────────────────┘    │
│  ┌─ Layer 6: Transients ────────────────────────┐    │
│  │  Shooting stars, comets, eclipses, satellites│    │
│  └──────────────────────────────────────────────┘    │
│  ┌─ Layer 7: Deep Space (optional) ─────────────┐    │
│  │  Distant galaxies, star clusters, dust lanes │    │
│  └──────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────┘
```

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
  } catch (e) {}
  if (!renderer) {
    renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    renderer.toneMappingExposure = 1.0;
  }
  renderer.setSize(innerWidth, innerHeight);
  renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
  return { renderer, gpuAvailable };
}
```

## Layer 1: Sky Dome

Gradient hemisphere that transitions from deep zenith to warm horizon glow, with
optional light pollution and twilight blending.

```javascript
function createSkyDome(radius = 500) {
  const geo = new THREE.SphereGeometry(radius, 32, 16);
  const material = new THREE.ShaderMaterial({
    uniforms: {
      zenithColor:   { value: new THREE.Color(0x020010) },
      midColor:      { value: new THREE.Color(0x0a0a2a) },
      horizonColor:  { value: new THREE.Color(0x15102a) },
      horizonGlow:   { value: new THREE.Color(0x1a1530) },
      glowStrength:  { value: 0.3 },
      lightPollution: { value: 0.0 },  // 0=pristine, 1=urban
      moonPos:       { value: new THREE.Vector3(0, 0.5, -1).normalize() },
      moonGlowStr:   { value: 0.15 },
    },
    vertexShader: `
      varying vec3 vWorldDir;
      void main() {
        vec4 worldPos = modelMatrix * vec4(position, 1.0);
        vWorldDir = normalize(worldPos.xyz);
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `,
    fragmentShader: SKY_DOME_FRAG, // See references/celestial-shaders.md
    side: THREE.BackSide,
    depthWrite: false,
  });

  return new THREE.Mesh(geo, material);
}
```

## Layer 2: Starfield

The heart of the night sky. Each star is a `Points` particle with scientifically-
grounded color (spectral class → blackbody temperature), magnitude-based size/brightness,
and animated twinkle.

### Star Generation

```javascript
class Starfield {
  constructor(scene, options = {}) {
    this.scene = scene;
    this.count = options.count ?? 8000;
    this.radius = options.radius ?? 400;
    this.minMagnitude = options.minMagnitude ?? -1.5; // Brightest (Sirius)
    this.maxMagnitude = options.maxMagnitude ?? 6.5;  // Faintest visible
    this.twinkleSpeed = options.twinkleSpeed ?? 1.0;
    this.seed = options.seed ?? 42;

    this._build();
  }

  _build() {
    const positions = new Float32Array(this.count * 3);
    const starData = new Float32Array(this.count * 4); // r, g, b, magnitude

    let rng = this.seed;
    const random = () => { rng = (rng * 16807) % 2147483647; return rng / 2147483647; };

    for (let i = 0; i < this.count; i++) {
      // Uniform distribution on sphere
      const theta = random() * Math.PI * 2;
      const phi = Math.acos(2 * random() - 1);

      positions[i * 3]     = Math.sin(phi) * Math.cos(theta) * this.radius;
      positions[i * 3 + 1] = Math.sin(phi) * Math.sin(theta) * this.radius;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CK42BB/procedural-stars-threejs](https://github.com/CK42BB/procedural-stars-threejs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: >
---


# Procedural Weather

Dynamic, layered weather effects in Three.js — GPU particle precipitation, volumetric
fog, lightning, and smooth state transitions.

## Architecture Overview

```
┌──────────────────────────────────────────────────────┐
│                Weather System                         │
│                                                      │
│  WeatherController (state machine)                   │
│    ├── current state + target state                  │
│    ├── transition progress (0→1)                     │
│    └── drives all subsystems:                        │
│                                                      │
│  ┌─ Precipitation ──────────────────────────────┐    │
│  │  GPU particles: rain, snow, hail             │    │
│  │  Ground splashes, accumulation               │    │
│  └──────────────────────────────────────────────┘    │
│  ┌─ Atmosphere ─────────────────────────────────┐    │
│  │  Fog, mist, dust, haze                       │    │
│  │  Volumetric (WebGPU) or exponential (WebGL)  │    │
│  └──────────────────────────────────────────────┘    │
│  ┌─ Electrical ─────────────────────────────────┐    │
│  │  Lightning bolts, sheet flashes              │    │
│  │  Cloud-internal illumination                 │    │
│  └──────────────────────────────────────────────┘    │
│  ┌─ Optical ────────────────────────────────────┐    │
│  │  Rainbows, aurora, god rays                  │    │
│  │  Wet lens, frost overlay                     │    │
│  └──────────────────────────────────────────────┘    │
│  ┌─ Environment ────────────────────────────────┐    │
│  │  Sky darkening, light color shift            │    │
│  │  Wind direction (shared across systems)      │    │
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
  }
  renderer.setSize(innerWidth, innerHeight);
  renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
  return { renderer, gpuAvailable };
}
```

## Shared Wind System

Wind drives all weather subsystems — precipitation angle, fog drift, debris direction.

```javascript
class WindSystem {
  constructor() {
    this.direction = new THREE.Vector3(1, 0, 0.3).normalize();
    this.baseSpeed = 5;       // m/s
    this.gustSpeed = 0;
    this.gustFrequency = 0.5;
    this.turbulence = 0.1;
    this._time = 0;
  }

  update(dt) {
    this._time += dt;
    // Gusts: low-frequency intensity variation
    const gustEnvelope = Math.sin(this._time * this.gustFrequency) * 0.5 + 0.5;
    this.gustSpeed = gustEnvelope * this.baseSpeed * 0.6;
  }

  get speed() { return this.baseSpeed + this.gustSpeed; }

  // Wind force vector for particle displacement
  get force() {
    const s = this.speed;
    const turb = new THREE.Vector3(
      Math.sin(this._time * 2.3) * this.turbulence,
      0,
      Math.cos(this._time * 1.7) * this.turbulence
    );
    return this.direction.clone().multiplyScalar(s).add(turb);
  }
}
```

## Precipitation System

GPU particle system for rain, snow, and hail. Particles spawn in a box above the
camera, fall with gravity + wind, and recycle when below ground.

### Particle Buffer Layout

```javascript
function createPrecipitationBuffers(maxParticles) {
  // Each particle: vec4(x, y, z, life) + vec4(vx, vy, vz, size)
  const positions = new Float32Array(maxParticles * 4);
  const velocities = new Float32Array(maxParticles * 4);
  return { positions, velocities, count: maxParticles };
}
```

### Rain System

```javascript
class RainSystem {
  constructor(scene, options = {}) {
    this.scene = scene;
    this.count = options.count ?? 50000;
    this.spawnRadius = options.spawnRadius ?? 40;
    this.spawnHeight = options.spawnHeight ?? 30;
    this.dropLength = options.dropLength ?? 0.3;
    this.intensity = options.intensity ?? 1.0; // 0-1

    this._buildGeometry();
  }

  _buildGeometry() {
    // Each raindrop: a short line segment (2 vertices)
    const positions = new Float32Array(this.count * 6); // 2 verts × xyz
    const randoms = new Float32Array(this.count * 2);   // per-drop seed + phase

    for (let i = 0; i < this.count; i++) {
      const x = (Math.random() - 0.5) * this.spawnRadius * 2;
      const y = Math.random() * this.spawnHeight;
      const z = (Math.random() - 0.5) * this.spawnRadius * 2;

      // Top vertex
      positions[i * 6]     = x;
      positions[i * 6 + 1] = y;
      positions[i * 6 + 2] = z;
      // Bottom vertex (offset by drop length)
      positions[i * 6 + 3] = x;
      positions[i * 6 + 4] = y - this.dropLength;
      positions[i * 6 + 5] = z;

      randoms[i * 2]     = Math.random();     // seed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CK42BB/procedural-weather-threejs](https://github.com/CK42BB/procedural-weather-threejs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: >
---


# Procedural Grass

Generate dense, animated, visually rich procedural grass in Three.js with a WebGPU-first
pipeline and automatic WebGL2 fallback.

## Architecture Overview

```
┌──────────────────────────────────────────────────────┐
│                   Grass Pipeline                      │
│                                                      │
│  1. Blade Geometry ── bezier-curved triangle strip    │
│  2. Placement ─────── terrain-aware scatter + density │
│  3. Instancing ────── InstancedMesh / storage buffer  │
│  4. Wind ──────────── layered noise displacement      │
│  5. Shading ───────── SSS approx + color variation    │
│  6. LOD ───────────── density fade + blade simplify   │
│  7. Interaction ───── radial push from world objects  │
├──────────────────────────────────────────────────────┤
│  WebGPU path: compute placement + storage buffers     │
│  WebGL path:  CPU placement + InstancedMesh           │
└──────────────────────────────────────────────────────┘
```

## Blade Geometry

Each grass blade is a **tapered triangle strip** shaped along a quadratic bezier curve.
This gives natural curvature with minimal vertex count.

### Blade Mesh Generator

```javascript
function createBladeGeometry(segments = 4, width = 0.06, height = 1.0, curvature = 0.3) {
  // segments+1 cross-sections, 2 verts each, plus 1 tip vertex
  const vertCount = (segments + 1) * 2 + 1;
  const positions = new Float32Array(vertCount * 3);
  const uvs = new Float32Array(vertCount * 2);
  const indices = [];

  for (let i = 0; i <= segments; i++) {
    const t = i / segments;
    // Quadratic bezier: p0=(0,0), p1=(curvature, 0.5), p2=(0, 1)
    const x = 2 * (1 - t) * t * curvature;           // lateral curve
    const y = t * height;                              // vertical
    const w = width * (1 - t * 0.8);                   // taper

    const vi = i * 2;
    // Left vertex
    positions[(vi) * 3]     = x - w * 0.5;
    positions[(vi) * 3 + 1] = y;
    positions[(vi) * 3 + 2] = 0;
    uvs[(vi) * 2]     = 0;
    uvs[(vi) * 2 + 1] = t;
    // Right vertex
    positions[(vi + 1) * 3]     = x + w * 0.5;
    positions[(vi + 1) * 3 + 1] = y;
    positions[(vi + 1) * 3 + 2] = 0;
    uvs[(vi + 1) * 2]     = 1;
    uvs[(vi + 1) * 2 + 1] = t;
  }

  // Tip vertex
  const tipIdx = (segments + 1) * 2;
  const tipX = 2 * 0.5 * 0.5 * curvature; // t≈midpoint approximation
  positions[tipIdx * 3]     = curvature * 0.5;
  positions[tipIdx * 3 + 1] = height;
  positions[tipIdx * 3 + 2] = 0;
  uvs[tipIdx * 2]     = 0.5;
  uvs[tipIdx * 2 + 1] = 1.0;

  // Triangle strip indices
  for (let i = 0; i < segments; i++) {
    const a = i * 2, b = i * 2 + 1, c = (i + 1) * 2, d = (i + 1) * 2 + 1;
    indices.push(a, b, c, b, d, c);
  }
  // Tip triangles
  const lastL = segments * 2, lastR = segments * 2 + 1;
  indices.push(lastL, lastR, tipIdx);

  const geometry = new THREE.BufferGeometry();
  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  geometry.setAttribute('uv', new THREE.BufferAttribute(uvs, 2));
  geometry.setIndex(indices);
  geometry.computeVertexNormals();
  return geometry;
}
```

**Segment count guide**: 3 segments for distant LOD, 4–5 for mid-range, 6–8 for close-up hero grass.

## Instance Data Layout

Each blade instance stores placement and variation data packed into instance attributes.

```javascript
function createGrassInstanceData(count) {
  return {
    // vec4: x, y, z, rotation
    positionRotation: new Float32Array(count * 4),
    // vec4: scaleX, scaleY, tilt, colorVariation
    scaleAndVariation: new Float32Array(count * 4),
  };
}
```

## Placement System

### CPU Placement (WebGL path)

Scatter blades on terrain with density modulation, slope rejection, and jittered grid for
uniform distribution without clumping.

```javascript
function placeGrassOnTerrain({
  terrainSize, maxHeight, heightFn, noiseFn,
  density = 40,        // blades per unit² at max density
  minHeight = 0.05,    // normalized terrain height
  maxSlopeAngle = 0.6, // radians, reject steep slopes
  seed = 0,
} = {}) {
  const gridStep = 1 / Math.sqrt(density);
  const halfSize = terrainSize / 2;
  const instances = [];

  // Seeded random for reproducibility
  let rng = seed;
  const random = () => { rng = (rng * 16807 + 0) % 2147483647; return rng / 2147483647; };

  for (let gx = -halfSize; gx < halfSize; gx += gridStep) {
    for (let gz = -halfSize; gz < halfSize; gz += gridStep) {
      // Jitter within grid cell
      const x = gx + (random() - 0.5) * gridStep;
      const z = gz + (random() - 0.5) * gridStep;

      // Normalized terrain coordinates
      const nx = x / terrainSize + 0.5;
      const nz = z / terrainSize + 0.5;
      if (nx < 0 || nx > 1 || nz < 0 || nz > 1) continue;

      const h = heightFn(nx, nz);
      if (h < minHeight) continue;

      // Slope check via finite difference
      const eps = gridStep * 0.5;
      const hx = heightFn(nx + eps / terrainSize, nz);
      const hz = heightFn(nx, nz + eps / terrainSize);
      const slope = Math.atan(Math.sqrt((hx - h) ** 2 + (hz - h) ** 2) * maxHeight / eps);
      if (slope > maxSlopeAngle) continue;

      // Density modulation via noise (patches and bare spots)
      const densityNoise = noiseFn(x * 0.05, z * 0.05);
      if (densityNoise < -0.2) continue; // bare patches

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CK42BB/procedural-grass-threejs](https://github.com/CK42BB/procedural-grass-threejs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: How to use Leva controls to create interactive debug UI for experiments
---


# Leva Controls Setup

This project uses [Leva](https://github.com/pmndrs/leva) to create interactive debug UIs for controlling shader parameters, material properties, and scene configurations.

## Installation

Already included in `package.json`:

```bash
pnpm add leva
```

## Basic Usage

### Import

```typescript
import { useControls } from "leva";
// Or with additional utilities
import { useControls, button, folder as levaFolder } from "leva";
```

### Simple Controls

For basic boolean or simple value controls:

```typescript
const { debug, postEnabled } = useControls({
  debug: false,
  postEnabled: {
    value: true,
    label: "PostProcessing",
  },
});
```

### Numeric Controls with Range

```typescript
const { exposure, rotationX, lightIntensity } = useControls({
  rotationX: { value: 0, min: -Math.PI, max: Math.PI, step: 0.01 },
  exposure: { value: 2.8, min: -5, max: 5, step: 0.1 },
  lightIntensity: { value: 1.3, min: 0, max: 5, step: 0.01 },
});
```

### Vector Controls (Position/Direction)

Leva auto-detects `{x, y, z}` objects as 3D vectors:

```typescript
const { lightPos } = useControls({
  lightPos: { value: { x: -0.53, y: 0.33, z: 0.1 }, step: 0.01 },
});
```

### Color Controls

Leva auto-detects hex strings as colors:

```typescript
const { lightColor } = useControls({
  lightColor: { value: "#8c8c8c" },
});
```

### Dropdown/Select Controls

```typescript
const DITHER_METHODS = {
  None: "none",
  "Bayer 2x2": "bayer2x2",
  "Bayer 4x4": "bayer4x4",
  "Bayer 8x8": "bayer8x8",
} as const;

const { ditherMethod } = useControls({
  ditherMethod: {
    value: "bayer4x4",
    options: DITHER_METHODS,
    label: "Dithering",
  },
});
```

## Advanced Patterns

### Functional Form with Buttons

Use the functional form `useControls(() => ({...}))` when you need access to the `get` function for buttons:

```typescript
const [{ rotationX, exposure }] = useControls(() => ({
  rotationX: { value: 0, min: -Math.PI, max: Math.PI, step: 0.01 },
  exposure: { value: 5, min: -5, max: 5, step: 0.1 },
  "Copy Settings": button((get) => {
    const settings = {
      rotationX: get("rotationX"),
      exposure: get("exposure"),
    };
    navigator.clipboard.writeText(JSON.stringify(settings, null, 2));
    console.log("Settings copied to clipboard:", settings);
  }),
}));
```

### onChange Handlers for Immediate Updates

Use `onChange` when you need to update values immediately without re-renders:

```typescript
const turnProgressTarget = useRef(0);
const coverOpenTarget = useRef(0);

const { debugPageTurn } = useControls({
  topRotation: {
    value: 0,
    min: 0,
    max: 1,
    step: 0.01,
    label: "Cover open",
    onChange: (v: number) => {
      coverOpenTarget.current = v;
    },
  },
  pageTurn: {
    value: 0,
    min: 0,
    max: 1,
    label: "Page turn",
    onChange: (v: number) => {
      turnProgressTarget.current = v;
    },
  },
  debugPageTurn: false,
});
```

### Folder Organization

Group related controls into collapsible folders:

```typescript
import { folder as levaFolder, useControls } from "leva";

const { debugTarget } = useControls({
  DebugTextures: levaFolder({
    debugTarget: {
      value: "screen",
      options: ["screen", "baseColor", "debug", "all"],
      onChange: (value) => {
        window.history.pushState(
          {},
          "",
          window.location.pathname + "?debugTarget=" + value
        );
      },
      transient: false,
    },
  }),
});
```

### URL Sync Pattern

Sync control values with URL parameters for shareable states:

```typescript
function getInitialSelectedTexture(defaultTexture: string, textures: string[]) {
  const query =
    typeof window !== "undefined"
      ? new URLSearchParams(window.location.search).get("debugTarget") ||
        defaultTexture
      : defaultTexture;

  if (textures.includes(query)) {
    return query;
  }
  return defaultTexture;
}

const { debugTarget } = useControls({
  debugTarget: {
    value: getInitialSelectedTexture("screen", Object.keys(textures)),
    options: Object.keys(textures).concat("all"),
    onChange: (value) => {
      if (typeof window !== "undefined") {
        window.history.pushState(
          {},
          "",
          window.location.pathname + "?debugTarget=" + value
        );
      }
    },
    transient: false,
  },
});
```

## Integration with THREE.js Uniforms

### Pattern: Direct Uniform Updates

The standard pattern for syncing Leva controls with THREE.js uniforms:

```typescript
function Scene() {
  const { rotationX, lightPos, lightIntensity, lightColor } = useControls({
    rotationX: { value: 0, min: -Math.PI, max: Math.PI, step: 0.01 },
    lightPos: { value: { x: 0, y: -1.11, z: 0.87 }, step: 0.01 },
    lightIntensity: { value: 3.64, min: 0, max: 10, step: 0.01 },
    lightColor: { value: "#8c8c8c" },
  });

  // Create uniforms once
  const baseUniforms = useUniforms(() => ({
    rotationX: new THREE.Uniform(rotationX),
    lightPos: new THREE.Uniform(
      new THREE.Vector3(lightPos.x, lightPos.y, lightPos.z)
    ),
    lightIntensity: new THREE.Uniform(lightIntensity),
    lightColor: new THREE.Uniform(new THREE.Color(lightColor)),
  }));

  // Update uniforms each render (outside useFrame for reactive updates)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/ralph-gpu](https://github.com/vercel-labs/ralph-gpu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->

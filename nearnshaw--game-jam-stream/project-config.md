---
trigger: always_on
description: This reference documents common patterns, components, and systems used in Decentraland SDK 7 scenes based on example implementations.
---

# Decentraland SDK 7 Scenes Context7 Reference

This reference documents common patterns, components, and systems used in Decentraland SDK 7 scenes based on example implementations.

## Table of Contents
- [Scene Structure](mdc:#scene-structure)
- [Entity-Component System](mdc:#entity-component-system)
- [Component Reference](mdc:#component-reference)
- [UI System](mdc:#ui-system)
- - @Pontr- @Eent hts  @Trons
- @Scene Optimization
- @Restricted Actions
- @Testing Framework
- @Network Connections
- @Blockchain Operations

## Scene Structure

### Basic Project Structure
```
├── src/
│   ├── index.ts          # Main entry point
│   ├── components.ts     # Custom component definitions
│   ├── systems.ts        # Custom system implementations
│   ├── factory.ts        # Entity creation functions
│   ├── utils.ts          # Helper functions
│   └── ui.tsx            # UI definitions with React
├── package.json
└── tsconfig.json
```

### Main Entry Point
```typescript
// index.ts
import { engine } from '@dcl/sdk/ecs'
import { setupUi } from './ui'
import { mySystem } from './systems'

export function main() {
  // Add systems to the engine
  engine.addSystem(mySystem)
  
  // Initialize UI
  setupUi()
  
  // Create initial entities
  // ...
}
```

## Entity-Component System

### Creating Entities
```typescript
import { engine, Transform, MeshRenderer, MeshCollider } from '@dcl/sdk/ecs'
import { Vector3 } from '@dcl/sdk/math'

// Create a new entity
const entity = engine.addEntity()

// Add components to the entity
Transform.create(entity, { 
  position: Vector3.create(8, 1, 8),
  scale: Vector3.create(1, 1, 1)
})

// Add a visual mesh
MeshRenderer.setBox(entity) // Predefined shapes: setBox, setSphere, setPlane, etc.

// Add collision
MeshCollider.setBox(entity)
```

### Defining Custom Components
```typescript
// components.ts
import { Schemas, engine } from '@dcl/sdk/ecs'

// Define a component with properties
export const Spinner = engine.defineComponent('spinner', { 
  speed: Schemas.Number 
})

// Define a tag component (no properties)
export const Cube = engine.defineComponent('cube-id', {})
```

### Creating Systems
```typescript
// systems.ts
import { engine, Transform } from '@dcl/sdk/ecs'
import { Quaternion, Vector3 } from '@dcl/sdk/math'
import { Spinner } from './components'

// System that rotates entities with the Spinner component
export function circularSystem(dt: number) {
  // Query all entities with both Spinner and Transform components
  const entitiesWithSpinner = engine.getEntitiesWith(Spinner, Transform)
  
  for (const [entity, spinner, transform] of entitiesWithSpinner) {
    // Get a mutable reference to modify the component
    const mutableTransform = Transform.getMutable(entity)
    
    // Apply rotation based on the spinner speed and delta time
    mutableTransform.rotation = Quaternion.multiply(
      mutableTransform.rotation,
      Quaternion.fromAngleAxis(dt * spinner.speed, Vector3.Up())
    )
  }
}
```

## Component Reference

### Transform
```typescript
// Position, rotation, and scale
Transform.create(entity, {
  position: Vector3.create(x, y, z),
  rotation: Quaternion.fromEulerDegrees(x, y, z), // or Quaternion.create()
  scale: Vector3.create(x, y, z),
  parent: parentEntity // optional, for hierarchical transformations
})

// Update transform
const transform = Transform.getMutable(entity)
transform.position = Vector3.create(newX, newY, newZ)
```

### Mesh Rendering
```typescript
// Basic shapes
MeshRenderer.setBox(entity)
MeshRenderer.setSphere(entity)
MeshRenderer.setPlane(entity)

// Material
import { Material, MeshRenderer } from '@dcl/sdk/ecs'
import { Color4 } from '@dcl/sdk/math'

// PBR material
Material.setPbrMaterial(entity, {
  albedoColor: Color4.fromHexString("#FF0000"),
  metallic: 0.5,
  roughness: 0.5,
  // other properties: emissiveColor, reflectivityColor, etc.
})

// Basic material
Material.setBasicMaterial(entity, {
  diffuseColor: Color4.White()
})
```

### 3D Models
```typescript
import { GltfContainer } from '@dcl/sdk/ecs'

// Load a 3D model
GltfContainer.create(entity, {
  src: 'models/model.glb',
  visibleMeshesCollisionMask: ColliderLayer.CL_POINTER // Optional
})

// Check loading state
if (GltfContainerLoadingState.get(entity).currentState === LoadingState.FINISHED) {
  // Model is loaded
}
```

### Colliders
```typescript
import { MeshCollider, ColliderLayer } from '@dcl/sdk/ecs'

// Basic colliders
MeshCollider.setBox(entity)
MeshCollider.setSphere(entity)
MeshCollider.setPlane(entity)

// With specific collision layer
MeshCollider.setBox(entity, ColliderLayer.CL_PHYSICS)
```

### Text
```typescript
import { TextShape } from '@dcl/sdk/ecs'
import { Color4 } from '@dcl/sdk/math'

TextShape.create(entity, {
  text: 'Hello Decentraland',
  fontSize: 3,
  textColor: Color4.White(),
  outlineWidth: 0.1,
  outlineColor: Color4.Black(),
  width: 4,
  height: 2,
  textWrapping: true
})
```

### Billboard
```typescript
import { Billboard } from '@dcl/sdk/ecs'

// Makes an entity always face the camera
Billboard.create(entity)
```

### Audio
```typescript
import { AudioSource } from '@dcl/sdk/ecs'

AudioSource.create(entity, {
  audioClipUrl: 'sounds/mySound.mp3',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nearnshaw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

---
trigger: always_on
description: The ParticleDissolveEffect is a sophisticated Metal-based particle rendering system that creates a dissolve/dust effect animation. It uses GPU acceleration for both compute (particle physics) and rendering operations, with a centralized rendering engine that manages multiple particle effects efficiently.
---

# ParticleDissolveEffect Technical Documentation

## System Architecture Overview

The ParticleDissolveEffect is a sophisticated Metal-based particle rendering system that creates a dissolve/dust effect animation. It uses GPU acceleration for both compute (particle physics) and rendering operations, with a centralized rendering engine that manages multiple particle effects efficiently.

## Core Components Deep Dive

### 1. ParticleRenderingEngine (The Central Orchestrator)

This is the heart of the system - a singleton that manages the entire Metal rendering pipeline:

```swift
public final class ParticleRenderingEngine {
    public static let shared = ParticleRenderingEngine()
    public let device: MTLDevice
    private let commandQueue: MTLCommandQueue
    private let metalLibrary: MTLLibrary
    private let clearPipelineState: MTLRenderPipelineState
    private let layer: ParticleEventLayer
```

**Technical details:**
- **Metal Device**: The GPU interface for all Metal operations
- **Command Queue**: Serializes GPU commands for execution
- **Metal Library**: Contains compiled shader functions
- **Clear Pipeline State**: Pre-compiled pipeline for clearing render targets
- **ParticleEventLayer**: A CAMetalLayer that triggers render cycles

The engine uses a **deferred rendering approach**:
1. Subjects register themselves as needing updates
2. Updates are batched and processed in a single render cycle
3. This minimizes GPU state changes and improves performance

### 2. Surface Management System

The `ParticleRenderingSurfaceManager` implements a sophisticated texture atlas system:

```swift
private final class ParticleRenderingSurfaceManager {
    private var surfaces: [Int: ParticleRenderingSurface] = [:]
    private var scheduledDeallocations: [ParticleRenderingSurfaceAllocation] = []
```

**Key concepts:**
- **IOSurface-backed textures**: Allows sharing between Metal and Core Animation
- **Bin packing algorithm**: Efficiently packs multiple particle effects into shared textures
- **Double buffering**: Each allocation has two phases for smooth updates
- **Dynamic allocation**: Surfaces grow/shrink based on demand

**Surface allocation strategy:**
1. Try to fit in existing surfaces using ShelfPack algorithm
2. If no space, create new surface with optimal dimensions
3. Surfaces are power-of-2 sized (2048x2048 default) for GPU efficiency
4. Larger effects get custom-sized surfaces

### 3. Particle Physics System

Each particle has this data structure in GPU memory:

```metal
struct Particle {
    packed_float2 offsetFromBasePosition;  // Current position offset
    packed_float2 velocity;                // Velocity vector
    float lifetime;                        // Remaining lifetime
};
```

**Physics simulation (per frame):**
1. **Gravity**: Applied as `velocity.y += 120.0 * timeStep`
2. **Motion**: `position += velocity * timeStep`
3. **Lifetime decay**: `lifetime -= timeStep`
4. **Easing**: Particles ease in from right to left using a window function

### 4. Rendering Pipeline

The rendering uses a **two-pass approach**:

**Pass 1: Compute (Physics Update)**
```metal
kernel void dustEffectUpdateParticle(...) {
    // Update particle positions based on physics
    // Apply easing based on horizontal position
    // Decay lifetime
}
```

**Pass 2: Render**
```metal
vertex QuadVertexOut dustEffectVertex(...) {
    // Transform particle position to screen space
    // Calculate texture coordinates
    // Apply alpha based on lifetime
}

fragment half4 dustEffectFragment(...) {
    // Sample texture at UV coordinates
    // Multiply by alpha for fading
}
```

### 5. Coordinate System Transformations

The system handles multiple coordinate spaces:

1. **Particle Space**: (0,0) to (width, height) in pixels
2. **Layer Space**: UIKit coordinates with flipped Y
3. **Normalized Device Coordinates (NDC)**: (-1,-1) to (1,1) for Metal
4. **Texture Space**: (0,0) to (1,1) for UV mapping

**Transformation pipeline:**
```
Particle Space → Layer Space → NDC → Screen Space
```

### 6. Memory Management

**Shared Buffers**: Particle data uses shared memory (`MTLStorageModeShared`):
- CPU can write initial values
- GPU updates physics
- Minimizes memory copies

**Buffer sizing**: `particleCount * 4 * (4 + 1)` bytes
- 4 bytes per float
- 5 floats per particle (2D position, 2D velocity, lifetime)

### 7. Animation Lifecycle

1. **Initialization**:
   ```swift
   addItem(frame: CGRect, image: UIImage)
   ```
   - Creates `ParticleItem` with Metal texture
   - Allocates particle buffer
   - Starts display link

2. **Per-Frame Update**:
   ```swift
   updateParticleAnimation(deltaTime: Double)
   ```
   - Updates particle phases
   - Triggers GPU compute
   - Marks for rendering

3. **GPU Execution**:
   - Compute shader updates physics
   - Render shader draws particles
   - Results composited to IOSurface

4. **Cleanup**:
   - Particles removed when lifetime reaches 0
   - Display link stopped when empty
   - Surfaces deallocated

### 8. Performance Optimizations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ertembiyik/Haptics](https://github.com/ertembiyik/Haptics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

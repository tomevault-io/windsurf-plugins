---
trigger: always_on
description: Rules for working with rendering layers in @gravity-ui/graph
---


## Layer System Overview
Layers are fundamental to the Canvas rendering pipeline in @gravity-ui/graph. They organize drawable components and manage rendering order and performance. They ensure that elements like backgrounds, connections, blocks, and interactive elements are drawn in the correct sequence.

**Primary Goal: Extensibility:** Layers are the primary mechanism for extending the library's core functionality. If a user request involves adding new visual elements or behaviors without altering the base logic, creating a new custom layer is the preferred approach.

## Key Layer Concepts
- **Purpose:** Each layer is responsible for a specific aspect of the graph, which can include **rendering visual elements** (e.g., `BackgroundLayer`, `ConnectionLayer`) or **managing behavior and logic** (e.g., handling specific user interactions, managing non-visual state). A layer might not have a direct visual representation but still participate in the graph's lifecycle and logic.
- **Base Class:** `src/services/Layer.ts` (Exports `Layer`, `LayerProps`, `LayerContext`).
- **`LayerProps` Interface:** Defines configuration for a layer. Key properties include:
    - `canvas`: Configuration for the HTML5 Canvas element (optional).
        - `zIndex`: Stacking order.
        - `classNames`: CSS classes.
        - `respectPixelRatio`: (boolean, default: true) Whether the canvas should account for device pixel ratio for sharper rendering.
        - `transformByCameraPosition`: (boolean, default: false) Whether the canvas transform should automatically follow the camera's position and scale.
    - `html`: Configuration for the HTML overlay element (optional).
        - `zIndex`: Stacking order.
        - `classNames`: CSS classes.
        - `transformByCameraPosition`: (boolean, default: false) Whether the HTML element's transform should automatically follow the camera's position and scale via CSS `matrix()`.
    - `camera`: The `ICamera` instance.
    - `graph`: The main `Graph` instance.
    - `root`: The root HTML element where the layer will be attached.
- **`LayerContext` Interface:** Provides context information accessible within the layer (via `this.context`). Includes:
    - `graph`, `camera`, `constants`, `colors`, `graphCanvas`, `ctx`, `layer` (self-reference).
- **Location:** Core layer logic and specific layer implementations are found in `src/components/canvas/layers/` and potentially base classes/services like `src/services/Layer.ts`.
- **Rendering Order/Priority:** Layers are rendered/processed in a specific sequence determined by the main graph component. The order is crucial for visual correctness and logical flow (e.g., connections below blocks, interaction handlers processed before rendering). Typical order might be: Background -> Connections -> Blocks -> Groups -> Behavior/Interaction Layers -> Highlight Layers.
- **Performance:** Layers optimize rendering and processing by:
    - Drawing/processing only elements or logic relevant to that layer.
    - Potentially culling elements outside the current viewport.
    - Managing redraws or logic updates based on relevant state changes.

## Layer Lifecycle and DOM Operations

- **Layer Initialization Sequence:**
  1. **Creation (`constructor` → `init`)**: Layer is created, but not yet attached to DOM
  2. **Attachment (`attachLayer` → `afterInit`)**: Layer is attached to DOM
  3. **Updates (rendering cycle)**: Layer responds to changes
  4. **Detachment (`detachLayer` → `unmount`)**: Layer is removed from DOM

- **DOM Operation Timing:** Any operations that require the layer to be attached to the DOM (such as DOM manipulations, style injections, measurements, or accessing parent elements) must be performed in the `afterInit` method, not in `init` or the constructor.

- **Why This Matters:** During the creation phase, the root element may be undefined or not yet attached to the document. The `afterInit` method is guaranteed to be called after the layer is properly attached to the DOM, ensuring that DOM operations work correctly.

## HTML Rendering in Layers

The Layer class can create both Canvas and HTML elements for rendering:

```typescript
// Example of Layer creation with both Canvas and HTML elements
constructor(props: LayerProps) {
  super({
    canvas: {
      zIndex: 2,
      respectPixelRatio: true,
      classNames: ["no-user-select"],
    },
    html: {
      zIndex: 3,
      classNames: ["no-user-select"],
      transformByCameraPosition: true, // This enables HTML rendering with camera transformation
    },
    ...props,
  });
}
```

### HTML Layer Configuration

- **There is no dedicated "HTMLLayer" class**. Instead, HTML rendering is handled by setting the `html` property in the Layer constructor options.
- The `transformByCameraPosition: true` option is crucial for HTML elements that need to move with the camera.
- When this option is set, the layer adds the "layer-with-camera" class to the HTML element and subscribes to camera state changes.

### Camera Transformation

- HTML elements with `transformByCameraPosition: true` are transformed using a matrix transform based on camera state:
```typescript
this.html.style.transform = `matrix(${camera.scale}, 0, 0, ${camera.scale}, ${camera.x}, ${camera.y})`;
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gravity-ui/graph](https://github.com/gravity-ui/graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

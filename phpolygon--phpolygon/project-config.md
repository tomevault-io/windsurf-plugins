---
trigger: always_on
description: PHP-native game engine with AI-first authoring. This file governs how Claude Code
---

# CLAUDE.md — PHPolygon Engine

PHP-native game engine with AI-first authoring. This file governs how Claude Code
works in this repository. Read it fully before writing any code.

---

## Engine identity

**PHPolygon** is a standalone PHP-native game engine. The primary authoring tool
is Claude Code. Worlds, characters, and game logic are written entirely in PHP —
no external 3D modelling tools (Blender, Maya, etc.) and no imported model files
(.fbx, .obj, .gltf) are part of the workflow. Geometry is generated procedurally
from PHP code.

Render backends:
- **2D:** php-vio (primary) or OpenGL 4.1 via php-glfw / NanoVG (fallback)
- **3D:** php-vio, Vulkan via php-vulkan, Metal via MoltenVK, or OpenGL 4.1 via
  php-glfw — all behind a unified `Renderer3DInterface` / `RenderCommandList`

The engine auto-detects php-vio at startup. When available, it provides the window,
input, audio, 2D renderer, 3D renderer, and texture manager through a single
unified backend. When php-vio is not loaded, the engine falls back to php-glfw
(2D/3D) or php-vulkan (3D).

Games are built in separate repositories and require `phpolygon/phpolygon`
via Composer.

---

## Architecture decisions (settled — do not revisit without explicit instruction)

### ECS: Hybrid model
- **Entities** are PHP objects with a component array. They have identity and lifecycle.
- **Components** own *per-entity* behaviour: `onAttach()`, `onUpdate()`, `onDetach()`,
  `onInspectorGUI()`. They may hold data and per-entity logic.
- **Systems** own *cross-entity* logic: physics, collision, economy, pathfinding.
  A System iterates components across multiple entities.
- **Discipline rule:** never put cross-entity logic in a Component. Never put
  per-entity render or state logic in a System. When in doubt, ask which boundary
  the code crosses.

### Scene authoring: PHP-canonical / split
- **PHP is always the canonical source of truth** for scene structure (entities,
  components, configuration).
- **JSON is the intermediate format** for the Vue/NativePHP editor. It is generated
  from PHP and consumed by the editor. The editor writes JSON back; a bidirectional
  transpiler converts to/from PHP.
- **Runtime state** (save games, dynamic positions, live game state) is always JSON.
- Use PHP 8.x `#[Attribute]` annotations to drive serialisation via Reflection.
  New components must never implement manual `toJson()` / `fromJson()` methods —
  the serialiser handles this automatically.
- Scene PHP files are version-controlled as code. JSON files are derived artefacts.

### Render interface: Layered
- `RenderContextInterface` — base: `beginFrame()`, `endFrame()`, `clear()`,
  `setViewport()`
- `Renderer2DInterface extends RenderContextInterface` — 2D backend (VioRenderer2D or NanoVG Renderer2D)
- `Renderer3DInterface extends RenderContextInterface` — 3D backend (Vio, Vulkan, Metal, or OpenGL)

`Renderer3DInterface` is driven by a **RenderCommandList** from day one. PHP builds
the command list; the backend executes it. This keeps game code fully backend-agnostic:

```
Game Code / Scene
      ↓  (builds)
RenderCommandList        ← pure PHP data, no GPU calls
      ↓  (executed by)
┌───────────────┬──────────────────┬──────────────────┬──────────────────┬──────────────────┐
VioRenderer3D   OpenGLRenderer3D   VulkanRenderer3D   MetalRenderer3D   NullRenderer3D
(primary)       (fallback)         (native Vulkan)    (MoltenVK/macOS)  (headless/tests)
```

**Do not design `Renderer3DInterface` around the OpenGL state-machine model.**
The OpenGL 3D backend *emulates* command buffers — it iterates the command list
and issues the necessary GL calls internally. Vulkan natively maps to this model.

### RenderCommandList — available commands

All commands are plain PHP value objects (no methods, only constructor properties):

| Command class | Purpose |
|---|---|
| `SetCamera` | `viewMatrix: Mat4`, `projectionMatrix: Mat4` |
| `SetAmbientLight` | `color: Color`, `intensity: float` |
| `SetDirectionalLight` | `direction: Vec3`, `color: Color`, `intensity: float` |
| `AddPointLight` | `position: Vec3`, `color: Color`, `intensity: float`, `radius: float` |
| `DrawMesh` | `meshId: string`, `materialId: string`, `modelMatrix: Mat4` |
| `DrawMeshInstanced` | `meshId: string`, `materialId: string`, `matrices: Mat4[]` |
| `SetSkybox` | `cubemapId: string` |
| `SetFog` | `color: Color`, `near: float`, `far: float` |
| `SetShader` | `shaderId: ?string` — override active shader for subsequent draws; `null` resets to material-driven |

Commands are appended to `RenderCommandList` during the scene tick. The
`Renderer3DSystem` flushes the list once per frame.

### Shader management

Games control shaders via the `Shader` facade or `$engine->shaders`:

```php
use PHPolygon\Support\Facades\Shader;

Shader::available();           // ['default', 'unlit', 'normals', 'depth', 'shadow', 'skybox']
Shader::use('unlit');          // global override — all draws use 'unlit'
Shader::active();              // 'unlit'
Shader::isOverridden();        // true
Shader::reset();               // back to material-driven selection

// Register a custom shader
Shader::register('toon', new ShaderDefinition(
    'resources/shaders/source/toon.vert.glsl',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phpolygon/phpolygon](https://github.com/phpolygon/phpolygon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

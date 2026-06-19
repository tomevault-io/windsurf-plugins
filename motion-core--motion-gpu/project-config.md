---
trigger: always_on
description: Build and edit MotionGPU code across framework-agnostic core and Svelte/React/Vue adapters. Use when implementing or refactoring FragCanvas-based components, defineMaterial shaders, useFrame runtime logic, textures/useTexture workflows, render passes/targets, compute shaders/storage buffers, render-mode scheduling, or MotionGPU error handling and diagnostics.
---


# MotionGPU Core + Adapters Skill

Use this skill to produce production-grade MotionGPU code across:
- framework-agnostic core (`@motion-core/motion-gpu`, `@motion-core/motion-gpu/core`),
- Svelte adapter (`@motion-core/motion-gpu/svelte`),
- React adapter (`@motion-core/motion-gpu/react`),
- Vue adapter (`@motion-core/motion-gpu/vue`).

Treat Svelte, React, and Vue as first-class adapters. Do not assume Svelte-only APIs.

## Source of Truth

Treat public package entrypoints as authoritative:

| Entrypoint | Layer | What it exposes |
| --- | --- | --- |
| `@motion-core/motion-gpu` | Core | Framework-agnostic runtime primitives (`defineMaterial`, `resolveMaterial`, scheduler/runtime builders, passes, texture loader, error normalization) |
| `@motion-core/motion-gpu/advanced` | Core | Core + scheduler helpers (`applySchedulerPreset`, `captureSchedulerDebugSnapshot`) |
| `@motion-core/motion-gpu/core` | Core | Same core API surface as root, explicit core path |
| `@motion-core/motion-gpu/core/advanced` | Core | Same advanced core helper surface |
| `@motion-core/motion-gpu/svelte` | Adapter | Svelte `FragCanvas`, hooks (`useMotionGPU`, `useFrame`, `usePointer`, `useTexture`), passes, material helpers |
| `@motion-core/motion-gpu/svelte/advanced` | Adapter | Svelte adapter + user context APIs + scheduler helpers |
| `@motion-core/motion-gpu/react` | Adapter | React `FragCanvas`, hooks (`useMotionGPU`, `useFrame`, `usePointer`, `useTexture`), passes, material helpers |
| `@motion-core/motion-gpu/react/advanced` | Adapter | React adapter + user context APIs + scheduler helpers |
| `@motion-core/motion-gpu/vue` | Adapter | Vue `FragCanvas`, composables (`useMotionGPU`, `useFrame`, `usePointer`, `useTexture`), passes, material helpers |
| `@motion-core/motion-gpu/vue/advanced` | Adapter | Vue adapter + user context APIs + scheduler helpers |

Advanced adapter exports:
- Svelte, React, and Vue advanced entrypoints export:
  - `useMotionGPUUserContext`
  - `setMotionGPUUserContext`
  - `applySchedulerPreset`
  - `captureSchedulerDebugSnapshot`
- React advanced additionally exports:
  - `useSetMotionGPUUserContext`

Import only from public entrypoints above. Never import from internal package paths (`/src`, `/lib/core`, etc.).

Documentation sources:
- LLM docs index: `http://motion-gpu.dev/llms.txt`
- Docs generated from source live under `apps/web/src/routes/docs`

If examples conflict with exported runtime behavior, prefer exported API contracts from entrypoints.

## Adapter Differences (Must Be Preserved)

When writing or refactoring code, keep these differences explicit.

### `FragCanvas` props

Shared runtime props (all adapters):
- `material`, `renderTargets`, `passes`, `clearColor`, `color`, `renderMode`, `autoRender`, `maxDelta`, `dpr`, `adapterOptions`, `deviceDescriptor`, `showErrorOverlay`, `onError`, `errorHistoryLimit`, `onErrorHistory`

Adapter-specific differences:
- Svelte:
  - `class?: string`
  - `style?: string`
  - `children?: Snippet`
  - `errorRenderer?: Snippet<[MotionGPUErrorReport]>`
- React:
  - `className?: string`
  - `style?: React.CSSProperties`
  - `children?: ReactNode`
  - `errorRenderer?: (report: MotionGPUErrorReport) => ReactNode`
- Vue:
  - `canvasClass?: string`
  - `canvasStyle?: string | Record<string, string | number>`
  - default slot for children
  - `#errorRenderer="{ report }"` scoped slot for custom error UI

### User context writes

- All adapters support `setMotionGPUUserContext(namespace, valueOrFactory, options?)`.
- React additionally supports `useSetMotionGPUUserContext()` and should prefer it for effect/event-handler writes.
- `SetMotionGPUUserContextOptions` supports:
  - `existing?: 'skip' | 'replace' | 'merge'`
  - `functionValue?: 'factory' | 'value'`

### `useTexture` signature

- Shared return shape: `{ textures, loading, error, errorReport, reload }`
- Shared URL input: `string[] | () => string[]`
- Options input:
  - Svelte: `TextureLoadOptions | () => TextureLoadOptions`
  - React: `TextureLoadOptions`
  - Vue: `TextureLoadOptions | () => TextureLoadOptions`

### `usePointer` signature

- Shared return shape: `{ state, lastClick, resetClick }`
- Shared option highlights:
  - `requestFrame?: 'auto' | 'invalidate' | 'advance' | 'none'`
  - `capturePointer?: boolean`
  - `trackWhilePressedOutsideCanvas?: boolean`
  - click synthesis options (`clickEnabled`, `clickMaxDurationMs`, `clickMaxMovePx`, `clickButtons`)
  - callbacks: `onMove`, `onDown`, `onUp`, `onClick`
- Coordinate conventions:
  - `state.current.uv` uses shader-friendly Y-up (`0..1`)
  - `state.current.ndc` uses Y-up (`-1..1`)

## Hard Contracts

Enforce these constraints without exceptions:

1. Material shader entrypoint must be exactly:
`fn frag(uv: vec2f) -> vec4f`
2. `ShaderPass` shader entrypoint must be exactly:
`fn shade(inputColor: vec4f, uv: vec2f) -> vec4f`
3. `ComputePass` shader must contain `@compute @workgroup_size(...)` and a `fn compute(...)` entrypoint.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Motion-Core/motion-gpu](https://github.com/Motion-Core/motion-gpu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

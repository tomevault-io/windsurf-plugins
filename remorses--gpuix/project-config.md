---
trigger: always_on
description: **Read [README.md](./README.md) first** to understand what GPUIX is, the architecture, mutation API, event flow, supported elements/events/styles, and the test renderer.
---

# AGENTS.md - GPUIX Codebase Guide

**Read [README.md](./README.md) first** to understand what GPUIX is, the architecture, mutation API, event flow, supported elements/events/styles, and the test renderer.

## Project Goal

GPUIX enables building **native GPU-accelerated desktop applications** using **React and TypeScript**, powered by [GPUI](https://github.com/zed-industries/zed/tree/main/crates/gpui) (Zed's rendering framework).

Instead of Electron/web rendering, your React components render directly to the GPU via Metal/Vulkan.

```
React (TypeScript)  →  napi-rs  →  GPUI (Rust)  →  GPU
     Your code         Bridge      Native render    Metal/Vulkan
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  JavaScript / TypeScript                                        │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Your React App                                          │   │
│  │                                                          │   │
│  │  function App() {                                        │   │
│  │    const [count, setCount] = useState(0)                 │   │
│  │    return (                                              │   │
│  │      <div style={{ display: 'flex', bg: '#1e1e2e' }}>    │   │
│  │        <div onClick={() => setCount(c => c + 1)}>+</div> │   │
│  │      </div>                                              │   │
│  │    )                                                     │   │
│  │  }                                                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  @gpuix/react (packages/react)                           │   │
│  │                                                          │   │
│  │  - React Reconciler (react-reconciler)                   │   │
│  │  - Builds element tree from React components             │   │
│  │  - Serializes to JSON ElementDesc                        │   │
│  │  - Manages event handler registry                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ↓ JSON                             │
└─────────────────────────────────────────────────────────────────┘
                               ↓ napi-rs FFI
┌─────────────────────────────────────────────────────────────────┐
│  Rust / Native                                                  │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  @gpuix/native (packages/native)                         │   │
│  │                                                          │   │
│  │  - GpuixRenderer: receives JSON, triggers re-render      │   │
│  │  - build_element(): ElementDesc → GPUI elements          │   │
│  │  - apply_styles(): StyleDesc → GPUI style methods        │   │
│  │  - Event handlers → ThreadsafeFunction callbacks to JS   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  GPUI (from zed)                                         │   │
│  │                                                          │   │
│  │  - Immediate-mode UI framework                           │   │
│  │  - Flexbox layout via Taffy                              │   │
│  │  - GPU rendering via Metal (macOS) / Vulkan (Linux)      │   │
│  │  - Native window management                              │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Key Insight: Immediate Mode Alignment

GPUI is **immediate-mode** - it rebuilds the entire UI tree every frame. This actually aligns perfectly with React's model:

| Traditional DOM Renderer | GPUIX |
|--------------------------|-------|
| `appendChild(node)` | Rebuild tree each render |
| `node.style.color = x` | Send full tree description |
| Mutation-based | Description-based |

We don't fight GPUI's architecture - we embrace it by sending a complete element description on every React render.

## Package Structure

```
gpuix/
├── packages/
│   ├── native/                 # Rust napi-rs bindings
│   │   ├── src/
│   │   │   ├── lib.rs          # Module exports
│   │   │   ├── renderer.rs     # GpuixRenderer, GpuixView, build_element()
│   │   │   ├── element_tree.rs # ElementDesc, EventPayload types
│   │   │   └── style.rs        # StyleDesc, color parsing
│   │   ├── examples/
│   │   │   └── hello.rs        # Pure GPUI test (no JS)
│   │   ├── Cargo.toml
│   │   └── build.rs
│   │
│   └── react/                  # React reconciler
│       ├── src/
│       │   ├── index.ts        # Public exports
│       │   ├── reconciler/
│       │   │   ├── host-config.ts  # React reconciler implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remorses/gpuix](https://github.com/remorses/gpuix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

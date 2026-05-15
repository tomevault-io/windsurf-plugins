---
trigger: always_on
description: **Core Framework**: Modern C++20 Vulkan 3D Engine.
---

# Emeraude Engine - AI Context

## 1. Context

**Core Framework**: Modern C++20 Vulkan 3D Engine.
**Coordinates**: **Right-Handed Y-DOWN** (Consistent across Physics, Rendering, Audio).
**Platform**: Windows 11, Linux (Debian/Ubuntu), macOS.
**Graphics API**: **Vulkan-only** — no D3D11, no D3D12, no Metal, no OpenGL. Single backend, mastered in full.

## 1b. Vision

### Unreal Engine 5 Runtime Killer

Emeraude Engine targets **surpassing Unreal Engine 5 runtime visual quality**. This is not
about the ecosystem (no editor, no blueprint system, no marketplace). The target is the
**runtime** — the core that runs on every end-user machine.

UE5's runtime consists of: renderer (Nanite, Lumen, virtual shadow maps, TSR), post-process
stack, material system, audio engine, physics. That is our battleground. If the developer
using emeraude-engine has to work harder on tooling — that is an acceptable trade-off. The
runtime output quality is the non-negotiable imperative.

### Strategic Positioning

"UE5 Runtime Killer" is a **direction**, not a frontal war. The real goal: give indie
developers a **genuine choice**. Today, a dev who wants AAA-quality rendering is locked
into UE5's licensing (5% royalties, Epic controls the terms) or Unity's unpredictable
pricing. Emeraude Engine offers a third option: **professional-grade runtime, LGPLv3,
zero royalties, zero contractual dependency.**

The trade-off is honest: less tooling, more code. The target audience is developers who
**want** to write C++ and understand their engine — not drag-and-drop users. Those who
accept this trade-off get a runtime that owes nothing to anyone.

### Open-Core Business Model

The engine follows an **open-core** model:
- **Runtime (emeraude-engine):** LGPLv3, free forever. The community's property.
- **Studio tooling (proprietary):** Scene editors, asset pipelines, productivity
  wrappers — paid products built on top of the free runtime.

The distinction is critical: **paid tools add convenience, never functionality.** A
developer who never pays has the exact same runtime capabilities. No feature gating,
no crippled free tier, no bait-and-switch. This model funds continued development of
the free runtime while keeping it permanently open.

### Vulkan-Only by Design

Emeraude Engine is **Vulkan-only**. This is a deliberate architectural decision, not a limitation.

UE5 maintains D3D11, D3D12, Vulkan, Metal — each backend is a compromise. They design
abstractions to the lowest common denominator. Emeraude Engine speaks directly to the GPU
through Vulkan, the open standard maintained by the **Khronos Group**. No abstraction layer,
no backend switching, no compromise.

This means:
- **Zero backend abstraction overhead** — the code talks directly to the GPU
- **Vulkan render passes, subpasses, layout transitions** are first-class citizens, not wrapped
- **Explicit synchronization** — full control over GPU scheduling
- **One target to optimize** — every microsecond gained benefits 100% of users
- **Khronos Group standard** — industrial-grade, cross-platform, vendor-neutral

Vulkan runs everywhere: Linux, Windows, Android, macOS/iOS via MoltenVK. The engine must be
implementable as a **Khronos Group showcase application**.

### AI-Driven Development

Emeraude Engine is developed **with AI and for AI**. The human is the **architect and director**.
The AI is the **implementor and analyst**.

This means:
- **The codebase must be AI-readable** — clear naming, consistent patterns, documented contracts
- **AI diagnostic tools are first-class** — RenderDoc integration, programmatic GPU analysis,
  automated visual regression testing are part of the engine, not external afterthoughts
- **Every rendering decision must be measurable** — frame capture, draw call counts, render pass
  structure, vertex throughput. No blind optimization, no guesswork.
- **The AI must be able to autonomously diagnose rendering issues** — capture a frame, analyze
  the GPU pipeline, identify bottlenecks, and propose solutions backed by data

This is a new model of engine development where the human defines the vision and architecture,
and the AI executes, measures, and iterates at industrial speed.

### AI Runtime Control — GOLD RULE

> [!CRITICAL]
> **The engine has a Remote Console (TCP port 7777).** Any AI working on this project
> **MUST** use it. This is not optional — it is the primary tool for understanding what
> the engine is doing at runtime.
>
> **Cross-platform tool (required on Windows):** Use `tools/remote-console.py` — works on Windows, Linux, and macOS:
> ```bash
> python3 tools/remote-console.py "COMMAND"
> ```
> On Linux/macOS, `nc` (netcat) also works:
> ```bash
> # Linux (use -q flag):
> echo "COMMAND" | nc -q 2 localhost 7777
>
> # macOS (use -w flag — macOS nc does not support -q):
> echo "COMMAND" | nc -w 2 localhost 7777
> ```
>
> **When the user asks "what's on screen?"** → take a screenshot:
> ```bash
> python3 tools/remote-console.py "Core.RendererService.screenshot()"
> ```
> Then **read the PNG file** to see the rendering output. You have eyes. Use them.
>
> **When you need to verify a rendering change** → screenshot before and after.
>
> **When you need to understand the scene** → query it:
> ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EmeraudeEngine/emeraude-engine](https://github.com/EmeraudeEngine/emeraude-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->

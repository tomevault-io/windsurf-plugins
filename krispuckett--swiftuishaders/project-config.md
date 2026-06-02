---
trigger: always_on
description: This file is for an AI agent (Claude Code, Codex, etc.) asked to **use, extend, modify, or rebuild** these shaders. It explains exactly how the package is wired so you can work without re-deriving the conventions.
---

# AGENTS.md: architecture for coding agents

This file is for an AI agent (Claude Code, Codex, etc.) asked to **use, extend, modify, or rebuild** these shaders. It explains exactly how the package is wired so you can work without re-deriving the conventions.

## What this package is

- A SwiftPM library, `SwiftUIShaders`, with **two source files**:
  - `Sources/SwiftUIShaders/Shaders/SwiftUIShaders.metal`: 41 `[[ stitchable ]]` SwiftUI fragment shaders plus 5 shared helper functions. This is the source of truth.
  - `Sources/SwiftUIShaders/ShaderEffects.swift`: one typed `View` modifier per shader, plus two private `ViewModifier`s (`_StaticShaderEffect`, `_AnimatedShaderEffect`) that do the actual `layerEffect` plumbing.
- No third-party dependencies. No app code. The `bcs_` prefix is just a namespace (it originated as "book cover shaders").

## How SwiftUI Metal shader effects work (the mental model)

SwiftUI exposes three shader entry points on `View`: `.colorEffect`, `.layerEffect`, and `.distortionEffect`. **Every shader here is a `layerEffect`**, because they all need to sample neighboring pixels (blur, displacement, refraction). A `layerEffect` Metal function has this fixed signature shape:

```metal
[[ stitchable ]] half4 bcs_name(
    float2 position,        // pixel being computed (SwiftUI supplies this)
    SwiftUI::Layer layer,   // the source view, sample it with layer.sample(coord) (SwiftUI supplies this)
    float2 size,            // view size in pixels  <- first user argument
    float time,             // seconds, for animation (only on animated shaders)
    float param1,           // ...effect-specific knobs
    ...
) { ... }
```

`position` and `layer` are provided by SwiftUI automatically. **Everything after `layer` is provided in order by the Swift call**, via `Shader.Argument` values (`.float`, `.float2`, `.color`, …). The arguments must match the Metal parameter order exactly.

At runtime the shaders live in a compiled `default.metallib` inside the package's resource bundle. You reach them with:

```swift
ShaderLibrary.bundle(.module).bcs_name(.float2(size), .float(t), .float(param1), ...)
```

`.module` resolves to the package bundle; `bcs_name` is a dynamic member that returns a `Shader`.

## The wrapper conventions (how `ShaderEffects.swift` maps to Metal)

For each Metal function, the generated `View` method maps parameters like this:

| Metal parameter | Swift wrapper does |
|---|---|
| `float2 position`, `SwiftUI::Layer layer` | nothing, SwiftUI supplies them |
| `float2 size` | passes `proxy.size` from `visualEffect`'s `GeometryProxy` |
| `float time` | passes elapsed seconds from `TimelineView(.animation)`; presence of `time` is what makes a shader "animated" |
| `float2 <point>` (e.g. `touchPos`) | exposes a `UnitPoint` (0...1) param, converts to pixels via `size` |
| any other `float` | exposes a typed `Float` param; default = midpoint of the documented `// lo-hi` range comment |

Two private modifiers carry the plumbing so the per-shader methods stay one-liners:

```swift
struct _StaticShaderEffect: ViewModifier {
    let maxSampleOffset: CGSize
    let shader: (CGSize) -> Shader
    func body(content: Content) -> some View {
        content.visualEffect { view, proxy in
            view.layerEffect(shader(proxy.size), maxSampleOffset: maxSampleOffset)
        }
    }
}

struct _AnimatedShaderEffect: ViewModifier {
    let maxSampleOffset: CGSize
    let shader: (CGSize, Float) -> Shader
    func body(content: Content) -> some View {
        TimelineView(.animation) { context in
            let t = Float(context.date.timeIntervalSinceReferenceDate
                .truncatingRemainder(dividingBy: 3600))   // bounded to keep float precision
            content.visualEffect { view, proxy in
                view.layerEffect(shader(proxy.size, t), maxSampleOffset: maxSampleOffset)
            }
        }
    }
}
```

> Why route through `ViewModifier` instead of calling `visualEffect` directly inside `TimelineView`? Because `TimelineView`'s `Content` generic can't be inferred from a bare `self.visualEffect { ... }` call. Inside `body(content:)`, `content` is a concrete `Content`, so inference succeeds. This is the load-bearing reason for the indirection. Don't "simplify" it away.

## How to add a new shader

1. **Write the Metal function** in `SwiftUIShaders.metal`. Follow the signature shape above. If it animates, include `float time`. Add a `// MARK: - Name` line and a one-line description comment above it, and `// lo-hi: meaning` comments on each parameter. The docs and defaults are derived from those.
2. **Declaration order matters.** A helper or function must be defined *before* anything that calls it. The shared helpers (`bcs_hash`, `bcs_valueNoise`, `bcs_fbm`, `bcs_hsb2rgb`) sit at the top; `bcs_sampleRegion` sits before its first user. Append new entry-point shaders; don't reorder existing ones.
3. **Add the Swift wrapper** in `ShaderEffects.swift`:

```swift
/// My Effect. What it does.
/// - Parameter strength: 0-10: how strong

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krispuckett/SwiftUIShaders](https://github.com/krispuckett/SwiftUIShaders) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->

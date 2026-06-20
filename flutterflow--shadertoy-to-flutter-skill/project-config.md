---
trigger: always_on
description: Convert Shadertoy GLSL fragment shaders into Flutter-compatible GLSL fragment shaders (.frag files) for use with Flutter's FragmentProgram / FragmentShader API, Impeller, or the material_palette "fill shader" / "wrap shader" pattern. Use whenever the user pastes a Shadertoy shader (or points to one on shadertoy.com) and wants to run it in Flutter, asks to port or adapt a fragment shader to Flutter, mentions making Shadertoy-style uniforms (iResolution, iTime, iMouse, iChannel*, mainImage, fragCo
---


# Shadertoy → Flutter shader converter

Convert a Shadertoy fragment shader into a Flutter-compatible `.frag` file that will compile and run under Flutter's `FragmentProgram`/`FragmentShader` API.

## The guiding principle

**Change only what Flutter's runtime forces you to change. Preserve everything else.**

Shadertoy shaders vary wildly — one might be a 3000-line raymarcher using three noise textures and mouse input; another might be a twelve-line procedural pattern with `sin` and `cos`. A good port leaves each shader's shape intact and only rewrites what would fail to compile under Flutter's restrictions.

That means:

- If the source doesn't sample `iChannel*`, **don't add any noise/hash helper functions to the output**. The shader already has its own procedural content — leave it alone.
- Preserve comments, preprocessor blocks (`#if`/`#else`/`#endif`, `#define`), commented-out alternative implementations, author attribution headers, and the author's variable names.
- Don't eagerly resolve preprocessor toggles to a single active branch. If there's a `#define LOOK 1` with `#if LOOK==0` / `#else` branches, keep both branches intact; translate any Shadertoy-isms inside each branch, but leave the toggle structure. The only exception is when an inactive branch uses a feature that can't be ported at all — then call it out in chat but still don't silently delete it.
- Don't "clean up" formatting, renumber things, or reshape the code. The user picked this shader because they like how it looks on Shadertoy; the Flutter version should look the same.

Your job is translation, not refactoring.

## Fill shader or wrap shader?

This is the one structural decision you have to make. It follows from how the source uses its channels:

- **No iChannel sampling at all** → fill shader. Nothing to wire up.
- **iChannel sampled as noise/dither** (patterns in "iChannel handling" below) → fill shader with procedural substitution. The presence of a sampler in the Shadertoy source does not promote it to wrap.
- **iChannel sampled as a scene/image/asset** the user wants to keep → wrap shader. Route through `uTexture`.
- **Multiple image channels** → at most one can become `uTexture`; see "iChannel handling" for how to handle the others.

When the channel's role is genuinely ambiguous from the code alone, ask the user once: "Is this meant to be a standalone visual, or an effect layered over your app UI / an image you supply?"

## The output template

Every output file, fill or wrap, starts with:

```glsl
#include <flutter/runtime_effect.glsl>
precision highp float;
```

**Do not emit `#version` directives.** Flutter injects its own version header during the offline GLSL → SPIR-V compile; writing your own triggers a duplicate-version error or a version mismatch depending on the toolchain. If you see `#version 460 core` in an older example, treat it as stale.

Then uniforms, in this order:

1. `uniform vec2 uSize;`
2. `uniform float uTime;`
3. Any additional non-sampler uniforms the port genuinely needs (see "Additional uniforms" below — default to *not* adding any)
4. `uniform sampler2D uTexture;` — **wrap shaders only**, always the last uniform

Then the output declaration:

```glsl
out vec4 fragColor;
```

Then the shader's helper functions, preserved verbatim except for targeted substitutions (see "Targeted substitutions" below).

Then `void main() { ... }`.

## Entry point rewrite

Shadertoy:
```glsl
void mainImage(out vec4 <outName>, in vec2 <inName>) {
    // ... body uses <outName> and <inName> ...
}
```

The parameter names vary — most shaders use `fragColor`/`fragCoord`, but iq's tiny raymarcher uses `c`/`p`, others use `outColor`/`uv`, etc. Preserve whatever names the original used inside the body; rebind them at the top of `main()`:

```glsl
void main() {
    vec2 <inName> = FlutterFragCoord().xy;
    vec4 <outName>;
    // ... original body, verbatim except for Shadertoy uniform substitutions ...
    fragColor = <outName>;
}
```

If `<outName>` is literally `fragColor`, just use the file-level `fragColor` directly and skip the local declaration + final assignment — it's already in scope.

If `<inName>` is literally `fragCoord`, just write `vec2 fragCoord = FlutterFragCoord().xy;` at the top.

This pattern lets you leave compact/obfuscated shaders (like short-char-count golfed ones) exactly as-is in the body.

## Targeted substitutions inside the body

These are the substitutions you apply everywhere in the body, including inside preprocessor branches (even inactive ones, for consistency) but **not** inside comments.

| Shadertoy identifier | Flutter replacement |
|----------------------|---------------------|
| `iTime` | `uTime` |
| `iResolution.xy` | `uSize` |
| `iResolution.x`, `iResolution.y`, `iResolution.z` | `uSize.x`, `uSize.y`, `1.0` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlutterFlow/shadertoy_to_flutter_skill](https://github.com/FlutterFlow/shadertoy_to_flutter_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: 1. **Single-line uniform declarations** (required for parsing tools)
---

# AS StageFX Shader Development Guide

## Critical Guidelines

1. **Single-line uniform declarations** (required for parsing tools)
   ```hlsl
   uniform float Param < ui_type = "slider"; ui_label = "Label"; ui_min = MIN; ui_max = MAX; ui_category = AS_CAT_APPEARANCE; > = DEFAULT;
   ```

2. **Technique guards** to prevent duplicate loading
   ```hlsl
   #ifndef __AS_LFX_ShaderName_1_fx
   #define __AS_LFX_ShaderName_1_fx
   // ...
   #endif // __AS_LFX_ShaderName_1_fx
   ```

3. **Consistent code organization**
   ```
   TECHNIQUE GUARD -> INCLUDES -> CONSTANTS -> UI DECLARATIONS ->
   HELPER FUNCTIONS -> PIXEL SHADER -> TECHNIQUE
   ```

4. **Use AS_Utils functions** instead of reimplementing common operations
   - Math constants: `AS_PI`, `AS_TWO_PI` (never use magic numbers)
   - Hash functions: `AS_hash11/21/12`
   - Noise functions: `AS_PerlinNoise2D`, `AS_Fbm2D`, etc.
   - Audio: `AS_audioModulate()`, `AS_audioLevelFromSource()`
   - Time: `AS_timeSeconds()`
   - Rotation: `AS_rotate2D()`
   - Compositing: `AS_composite()`, `AS_compositeRGBA()`
   - Color: `AS_adjustSaturation()`, `AS_srgbToLinear()`, `AS_linearToSrgb()`
   - UI macros: `AS_AUDIO_UI()`, `AS_BLENDMODE_UI()`, etc.

5. **Resolution independence** - All effects must render consistently regardless of screen size/ratio

6. **Namespace convention** - Wrap helper functions in a namespace:
   ```hlsl
   namespace AS_EffectName {
       // helpers and constants
   }
   ```
   Use the effect name without type prefix (e.g., `AS_BlueCorona`, not `AS_BGX_BlueCorona`).

7. **Shader descriptor** - Every shader must include `as_shader_descriptor` for attribution.

## Category Constants

Always use `AS_CAT_*` constants for `ui_category`. Never hardcode category strings.

| Constant | Value |
|----------|-------|
| `AS_CAT_PALETTE` | "Palette & Style" |
| `AS_CAT_APPEARANCE` | "Appearance" |
| `AS_CAT_PATTERN` | "Pattern" |
| `AS_CAT_LIGHTING` | "Lighting" |
| `AS_CAT_COLOR` | "Color" |
| `AS_CAT_ANIMATION` | "Animation" |
| `AS_CAT_AUDIO` | "Audio Reactivity" |
| `AS_CAT_STAGE` | "Stage" |
| `AS_CAT_PERFORMANCE` | "Performance" |
| `AS_CAT_FINAL` | "Final Mix" |
| `AS_CAT_DEBUG` | "Debug" |

## UI Standards

### Uniform Organization (canonical order)

1. Effect-specific categories (custom per shader)
2. `AS_CAT_PALETTE`
3. `AS_CAT_APPEARANCE`
4. `AS_CAT_ANIMATION`
5. `AS_CAT_AUDIO`
6. `AS_CAT_STAGE`
7. `AS_CAT_PERFORMANCE`
8. `AS_CAT_FINAL`
9. `AS_CAT_DEBUG`

### UI Macros

Standard macros for common controls:

- `AS_AUDIO_UI()` -- Audio source selector
- `AS_AUDIO_MULT_UI()` -- Audio multiplier slider
- `AS_AUDIO_TARGET_UI()` -- Audio target combo
- `AS_AUDIO_GAIN_UI()` -- Audio gain slider
- `AS_BLENDMODE_UI()` / `AS_BLENDMODE_UI_DEFAULT()` -- Blend mode selector
- `AS_COLOR_CYCLE_UI()` -- Color cycling toggle
- `AS_USE_PALETTE_UI()` -- Palette enable toggle
- `AS_BACKGROUND_COLOR_UI()` -- Background color picker

### Category Organization

- Group related parameters with `AS_CAT_*` constants
- Use `ui_category_closed = true` for secondary categories
- For multi-instance effects, use `ui_category = "Effect " #index`

## Compositing Guidance

BlendMode belongs to the user for the final mix. Never hardcode blend logic into the core effect computation. Instead:

1. Compute the effect result as clean RGB output
2. Use `AS_composite()` or `AS_compositeRGBA()` at the end to blend with the background
3. Expose `AS_BLENDMODE_UI()` so users control how the effect mixes

## Spatial Mode Guidance

- **BGX (Background)**: Full-screen procedural content. Uses stage depth to render behind the scene. Typically uses `AS_STAGE_DEPTH_UI` and position/scale controls.
- **VFX (Visual Effect)**: Overlay or post-process on existing scene. May use position/scale for placement. Does not replace the background.
- **LFX (Lighting)**: Light-based effects (spotlights, lasers). Uses position/rotation for light source placement.
- **GFX (Graphics Filter)**: Full-screen image processing (halftone, oil paint). Typically no position controls; processes entire frame.

## Noise Library

Noise functions are split across two headers:

- **`AS_Noise.1.fxh`** -- Core: `AS_hash*`, `AS_PerlinNoise2D`, `AS_Fbm2D`, `AS_VoronoiNoise2D`, `AS_DomainWarp2D`
- **`AS_Noise_Extended.1.fxh`** -- Advanced noise for specialized effects

Include only what you need. Most shaders only require `AS_Noise.1.fxh`.

## Pattern Implementation Guide

### Texture-Based Effects
```hlsl
// Preprocessor-based texture UI
#ifndef TEXTURE_PATH
#define TEXTURE_PATH "default.png"
#endif

texture EffectTexture < source = TEXTURE_PATH; ui_label = "Texture"; >
{ Width = 256; Height = 256; Format = RGBA8; };
sampler TextureSampler { Texture = EffectTexture; AddressU = REPEAT; AddressV = REPEAT; /* etc */ };
```

### Audio Reactivity - Parameter-Specific
```hlsl
// UI element for selecting which parameter reacts to audio
uniform int AudioTarget < ui_type = "combo"; ui_label = "Audio Target";
   ui_items = "None\0Parameter A\0Parameter B\0"; > = 0;

// In pixel shader
float paramA_final = paramA;
if (AudioTarget == 1) {
    float audioValue = AS_audioModulate(1.0, AudioSource, AudioMult, true, 0) - 1.0;
    paramA_final = paramA + (paramA * audioValue * scaleFactor);
}
```

### Rotated Distortion Effects

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeonAquitaine/as-stagefx](https://github.com/LeonAquitaine/as-stagefx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->

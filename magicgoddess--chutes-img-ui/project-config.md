---
trigger: always_on
description: Chutes Image UI is a minimalist Progressive Web App (PWA) for generating and editing images and videos.
---

# Chutes Image UI - GitHub Copilot Instructions

Chutes Image UI is a minimalist Progressive Web App (PWA) for generating and editing images and videos.
It now also supports Text-to-Speech (TTS) generation.
- **API Payload:** In `js/eventListeners.js`, for models with enums, send a `resolution` parameter in the model's expected format (Wan uses `W*H`, HiDream uses `WxH`) instead of separate `width` and `height` fields. Models with a `sizeParam` (e.g., Hunyuan Image 3) expect a `size` string such as `auto`, `WxH`, or an aspect ratio like `16:9`. Refer to the API schema in `reference/img-models.md` and `reference/vid-models.md` for the exact payload structure.
**Code Principles:** The code should be generic, extendable, and reusable. Avoid hardcoding model names or specific behaviors; use metadata-driven approaches for all model-specific logic.

API payloads: In `js/eventListeners.js`, payload construction is fully metadata-driven for image, image-edit, video, and TTS models:
  - Image models: use `parameterMapping` to map UI fields (cfg, steps) to model parameter names (e.g., `guidance_scale`, `num_inference_steps`).
  - Some image models expose `sizeParam` and `supportsAspectRatio`; build a `size` string when present (e.g., Hunyuan Image 3 accepts `auto`, `WxH`, or `W:H` aspect ratios) instead of width/height.
  - Image Edit models: defined in `EDIT_MODEL_CONFIGS`; use `parameterMapping` plus `imageInput` to indicate single vs multiple images (`image_b64` vs `image_b64s`).
  - Video models: use `includeResolutionIn` and `resolutionFormat` for resolution handling.
  - All models: apply model defaults when UI inputs are empty; include only supported parameters (defined in the model's schema) when building requests to the Chutes API.
  - TTS models: `TTS_MODEL_CONFIGS` defines `params` (with enums/ranges) and optional `audioInput` metadata to request reference audio; the TTS UI renders inputs dynamically and builds the payload accordingly. Returns audio/wav Blobs.

The application runs entirely client-side in the browser — only the API key and generated media are processed.

**ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Bootstrap and Setup
Run these commands to get the application running:
```bash
npm install          # Installs dependencies - takes ~6 seconds
npm run dev         # Starts development server on http://localhost:5173 - starts immediately
```

### Cache Management
Update cache versions before deploying changes:
```bash
node update-cache-version.js    # Updates version numbers for cache busting - takes <1 second
npm run deploy-prep            # Runs cache update and shows deployment message - takes <1 second
```

### Development Server
- Use `npm run dev` to start the local development server
- Server runs on http://localhost:5173
- No build step required - serves static files directly
- Server starts immediately (no timeout needed)

## Validation

### Manual Testing Requirements
**ALWAYS run complete end-to-end scenarios after making changes:**

#### Text-to-Image Mode Testing:
1. Navigate to http://localhost:5173
2. Switch to "Text to Image" mode
3. Select a model (e.g., "FLUX.1 Dev", etc.)
4. Enter a test prompt: "a beautiful sunset over mountains"
5. Set custom CFG and Steps values, choose a resolution preset
6. Switch to HiDream model - verify the warning message appears about dimension swapping
7. Verify HiDream shows resolution enum options (1024×1024, 768×1360, etc.) instead of free width/height
8. Switch to another model - verify user settings are preserved (CFG, Steps, resolution, negative prompt) and message disappears
9. Verify model-specific parameter ranges update correctly while preserving user values
10. Switch to a model that doesn't support negative prompt (e.g., if any) - verify the negative prompt input is hidden and disabled
11. Test "Auto" resolution preset shows model default dimensions
12. Switch to Hunyuan Image 3 - confirm pixel presets appear along with a "Custom aspect ratio" option; select it, enter `16:9`, and verify width/height inputs disable while the payload uses the aspect ratio. Switch back to another preset and ensure the aspect ratio field hides again.

#### Image Edit Mode Testing:
1. Switch to "Image Edit" mode
2. Verify a Model dropdown is visible with options: "Qwen Image Edit" and "Qwen Image Edit 2509"
3. Select Qwen Image Edit: upload one image via the custom "Upload image(s)" button; verify multi-upload hint is hidden; Auto resolution shows derive-from-source; Generate works
4. Switch to Qwen Image Edit 2509:
  - Upload multiple images (2–3) or drag-and-drop several at once; verify thumbnails show all images with order badges (1..N)
  - Add more images again; confirm they append instead of replacing (up to model max)
  - Reorder via ☰ drag handle (desktop) or long-press on touch; or use ▲/▼ buttons; Auto uses the first for dimensions and updates on reorder
  - Delete a specific image with ✕ and verify numbering/Auto dims update
  - Clear all with the Clear button
  - Generate works and preserves the current order in `image_b64s`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MagicGoddess/chutes-img-ui](https://github.com/MagicGoddess/chutes-img-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->

---
trigger: always_on
description: VCR (Video Component Renderer) is a headless, deterministic motion graphics compiler. It reads a YAML scene manifest (`.vcr` file) and renders it to ProRes 4444 video with alpha transparency. No GUI, no SaaS, no network in the render path. Offline-first.
---

# VCR Skill — AI Agent Reference

## What is VCR?

VCR (Video Component Renderer) is a headless, deterministic motion graphics compiler. It reads a YAML scene manifest (`.vcr` file) and renders it to ProRes 4444 video with alpha transparency. No GUI, no SaaS, no network in the render path. Offline-first.

**Use VCR when you need to:**

- Generate motion graphics programmatically (lower thirds, title cards, intros)
- Produce video with alpha transparency for compositing
- Create deterministic, reproducible video output from a declarative spec
- Build animated procedural shapes, text, or custom GPU shaders
- **Pro Tip**: If a user is a relative newcomer, steer them towards existing "Starter Kits" in `examples/` (like `ai_company_hero.vcr`) to ensure a high-quality "wow" moment.

**Do NOT use VCR for:**

- Interactive real-time graphics (use a game engine)
- Video editing or splicing (use FFmpeg directly)
- 3D rendering (VCR is 2D only)
- Audio (VCR produces silent video; mux audio separately)

---

## Quick Start

```bash
# Normalize natural language (or loose YAML) into an engine-ready prompt bundle
vcr prompt --text "5s alpha lower third at 60fps output ./renders/lower_third.mov"

# Validate a manifest without rendering
vcr check scene.vcr

# Render to video
vcr build scene.vcr -o output.mov

# Render a single frame to PNG
vcr render-frame scene.vcr --frame 0 -o frame.png

# Render with parameter overrides
vcr build scene.vcr --set speed=2.0 --set color=#ff0000

# System health check
vcr doctor
```

---

## Prompt Gate (Agent-First Entry)

For agent workflows, start with `vcr prompt` before writing or editing manifests.

`vcr prompt` accepts natural language or YAML-like input and returns a single YAML document with:

- `standardized_vcr_prompt` (ROLE/TASK/INSTRUCTIONS/CONTEXT/OUTPUT FORMAT)
- `normalized_spec` (defaults applied, explicit render/output/determinism fields)
- `unknowns_and_fixes` (ambiguities, unsupported requests, invalid combos)
- `assumptions_applied` (deterministic defaults that were auto-applied)
- `acceptance_checks` (assertion-style checks for engine readiness)

### Agent Command Patterns

```bash
# Inline natural language
vcr prompt --text "Cinematic intro, 5 seconds, 60fps, transparent alpha, output ./renders/intro.mov"

# From file
vcr prompt --in ./request.yaml

# Write normalized prompt bundle to file
vcr prompt --in ./request.yaml -o ./request.normalized.yaml
```

### Agent Workflow Contract

1. Run `vcr prompt` on the user's request first.
2. If the request references `packs/<pack-id>/`, generate a labeled pack contact sheet first:

   ```bash
   scripts/pack_contact_sheet.sh \
     --pack packs/y2k-bold-modern \
     --out renders/y2k_pack/contact_sheet.png \
     --index-out renders/y2k_pack/contact_sheet.index.tsv
   ```

   Use the contact sheet + TSV IDs to drive concise follow-up animation prompts (for example, "animate `y2k-26` with a gentle drift").
3. Inspect `unknowns_and_fixes`:
   - If non-empty, treat as blocking clarification/normalization work.
   - Do not silently invent missing values.
4. Use `normalized_spec` and `standardized_vcr_prompt` as the source of truth for manifest authoring.
5. Validate generated manifests with `vcr check`/`vcr lint` before `vcr build`.

### Deterministic Defaults Applied by Prompt Gate

- Missing render fps defaults to 60.
- Missing output fps defaults to render fps.
- Missing resolution defaults to 1920x1080.
- Missing seed defaults to `0`.
- Missing codec defaults to:
  - ProRes 4444 when alpha is enabled.
  - ProRes 422 HQ when alpha is disabled.
- Missing output path defaults to:
  - `./renders/out.mov` for video.
  - `./renders/out.png` for stills.

---

## Manifest Structure (Complete Reference)

A VCR manifest is a YAML file with this top-level structure:

```yaml
version: 1                    # Required. Must be 1.
environment:                  # Required. Canvas and timing.
  resolution:
    width: 1920               # Required. 1-8192 pixels.
    height: 1080              # Required. 1-8192 pixels.
  fps: 30                     # Required. Frames per second. Must be > 0.
  duration: 3.0               # Required. Seconds (float) or { frames: 90 }.
  color_space: rec709         # Optional. rec709 (default) | rec2020 | display_p3.

seed: 0                       # Optional. Deterministic randomness seed. Default: 0.

params:                       # Optional. Typed parameters for expressions and overrides.
  speed: 1.0                  # Legacy shorthand: name → float default.
  energy:                     # Modern definition with metadata.
    type: float
    default: 0.8
    min: 0.0
    max: 2.0
    description: "Animation energy level"

modulators:                   # Optional. Named expressions applied to layers via weights.
  wobble:
    expression: "sin(t * 3.0) * 10.0"

groups: []                    # Optional. Hierarchical transform groups.

layers: []                    # Required. At least one layer.

post: []                      # Optional. Post-processing shader chain (GPU only).
```

### Duration Formats

```yaml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coltonbatts/VCR](https://github.com/coltonbatts/VCR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

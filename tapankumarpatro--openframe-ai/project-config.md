---
trigger: always_on
description: Handles two kinds of audio:
---

# OpenFrame AI — Agent Pipeline

> How the 7 AI agents collaborate to create a video advertisement from a single text prompt.

---

## Pipeline Overview

```
User Input (product/brand description + ad type)
         │
         ▼
┌─────────────────────┐
│  Creative Director   │  → Campaign title, concept, mood, tagline
│  (+ Critic + Revise) │  → Self-critique loop for quality
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   Brand Stylist      │  → Color palette, textures, composition style
└─────────┬───────────┘
          │
    ┌─────┼─────────┐       (fan-out: 3 agents run in parallel)
    ▼     ▼         ▼
┌───────┐ ┌───────┐ ┌──────────────┐
│Product│ │Casting│ │Cinematographer│
│Stylist│ │ Scout │ │              │
└───┬───┘ └───┬───┘ └──────┬───────┘
    │         │            │
    └─────────┼────────────┘  (fan-in: all 3 must finish)
              ▼
┌─────────────────────┐
│      Director        │  → Shot list with scene breakdowns
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   Sound Designer     │  → Voiceover, music, per-scene dialogue
└─────────────────────┘
          │
          ▼
     Visual Canvas
  (user takes control)
```

---

## Agent Details

### 1. Creative Director

**File:** `src/agents/agent_1_creative.py`
**Output key:** `creative_brief`
**Schema:** `CreativeBrief`

Develops the campaign concept based on the user's product/brand description. Adapts tone to the product type:
- Luxury/fashion → abstract, elegant, aspirational
- Consumer/tech → clean, benefit-driven, trustworthy
- Social/UGC → raw, authentic, relatable
- Cinematic → story-driven, emotional, manifesto-style

**Output:**
```json
{
  "campaign_title": "String",
  "concept_summary": "String",
  "mood_keywords": ["String", "String"],
  "tagline": "String"
}
```

**Quality loop:** After the Creative Director produces a brief, it goes through a **Critic** (`agent_1_1_critique.py`) that identifies misalignments with the user's request, then a **Reviser** that fixes every flagged issue. This ensures the concept stays faithful to the original input.

---

### 2. Brand Stylist

**File:** `src/agents/agent_2_brand.py`
**Output key:** `visual_identity`
**Schema:** `VisualIdentity`
**Depends on:** Creative Director output

Defines the visual identity — colors, textures, composition — based on the campaign concept and mood.

**Output:**
```json
{
  "color_palette": ["Color 1", "Color 2", "Color 3"],
  "textures_materials": "String",
  "composition_style": "String"
}
```

---

### 3. Product Stylist

**File:** `src/agents/agent_3_product.py`
**Output key:** `product_specs`
**Schema:** `ProductSpecs`
**Depends on:** Brand Stylist output
**Runs in parallel with:** Casting Scout, Cinematographer

Rewrites the product description so an AI image generator understands its material quality, surface detail, and placement.

**Output:**
```json
{
  "material_behavior": "String",
  "surface_detail": "String",
  "styling_integration": "String",
  "visual_product_description": "String"
}
```

---

### 4. Casting Scout

**File:** `src/agents/agent_4_casting.py`
**Output key:** `casting_brief`
**Schema:** `CastingBrief`
**Depends on:** Brand Stylist output
**Runs in parallel with:** Product Stylist, Cinematographer

Defines the "Key Drivers" — cast members (human, animal, object, etc.) with full visual descriptions, and 2 distinct environments/settings.

**Output:**
```json
{
  "cast_members": [
    { "name": "Hero Model", "driver_type": "human", "visual_prompt": "..." }
  ],
  "setting_a": "String",
  "setting_b": "String"
}
```

---

### 5. Cinematographer

**File:** `src/agents/agent_5_cine.py`
**Output key:** `camera_specs`
**Schema:** `CameraSpecs`
**Depends on:** Brand Stylist output
**Runs in parallel with:** Product Stylist, Casting Scout

Creates the "Global Look" — lighting, camera gear, color temperature, and contrast. Generates a `technical_prompt_block` string that gets appended to every image generation prompt.

**Output:**
```json
{
  "lighting": "String",
  "camera_gear": "String",
  "color_temperature": "String",
  "contrast_tone": "String",
  "technical_prompt_block": "String"
}
```

---

### 6. Director

**File:** `src/agents/agent_6_director.py`
**Output key:** `shot_list`
**Schema:** `ShotList`
**Depends on:** Product Stylist + Casting Scout + Cinematographer (all three)

Assembles the scene sequence. Scene count adapts to the ad type (UGC: 1-3, Commercial: 5-8, Cinematic: 10-20). Each scene includes:
- Scene type (Intro, Reveal, Action, Closing, etc.)
- Shot type (Wide, Medium, Close-Up, Tracking, POV, etc.)
- Visual type (Standard, Model Shot, Product Shot, B-Roll, Glitch Art, etc.)
- Audio mode (silent, talking-head, audio-native)
- Start and end image prompts
- Three video prompts (start, end, combined)
- Dialogue and speaker (for non-silent scenes)

**Output:**
```json
{
  "scenes": [
    {
      "scene_number": 1,
      "type": "Intro",
      "shot_type": "Wide Shot",
      "visual_type": "Standard",
      "audio_mode": "silent",
      "action_movement": "String",
      "visual_description": "String",
      "start_image_prompt": "String",
      "end_image_prompt": "String",
      "start_video_prompt": "String",
      "end_video_prompt": "String",
      "combined_video_prompt": "String",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tapankumarpatro/openframe-ai](https://github.com/tapankumarpatro/openframe-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

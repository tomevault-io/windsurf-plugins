---
trigger: always_on
description: |
---


# AI Ad Prompt Guide

Universal prompting framework for AI video and image generation — works across all major models.

---

## Part 1: Universal Prompting Rules (Standalone)

### 1.1 The SLCT Framework

Every effective AI generation prompt has four components. Use SLCT as a checklist:

**S — Subject**: What is the main focus?
**L — Lighting/Look**: What's the visual mood?
**C — Camera**: What angle, movement, and framing?
**T — Technical**: Resolution, aspect ratio, duration, style keywords?

#### SLCT Examples

**Product B-roll:**
```
S: A glass bottle of amber serum on a marble bathroom counter
L: Soft golden morning light streaming from the left, creating gentle highlights on the glass
C: Slow push-in from medium to close-up, shallow depth of field
T: 4K, 5 seconds, photorealistic, product photography style
```

**UGC-style:**
```
S: A woman in her late 20s opening a package on her couch, looking excited
L: Natural indoor lighting, warm tones, slightly imperfect like a phone camera
C: Medium shot, handheld slight shake, selfie-style front camera angle
T: 9:16 vertical, 5 seconds, realistic, casual home setting
```

**Cinematic hero shot:**
```
S: A sleek electric car driving along a coastal highway at sunset
L: Dramatic golden hour, long shadows, warm highlights on the car body
C: Low-angle tracking shot from the front quarter, smooth dolly movement
T: 16:9, 8 seconds, cinematic, film grain, anamorphic lens flare
```

### 1.2 Hallucination Prevention

AI models hallucinate when prompts are ambiguous, contradictory, or physically impossible. These rules minimize bad outputs:

#### The 5 Rules of Hallucination Prevention

1. **Be spatially explicit**: "A bottle on the LEFT side of a marble counter, a plant on the RIGHT" — not "a bottle near a plant on a counter"

2. **Limit entities**: Maximum 3 main subjects per scene. More = more chance of merging/distortion

3. **Avoid negatives**: Don't say "no people in the background" — instead describe what IS there: "empty cafe with wooden chairs"

4. **Use real-world references**: "lighting like a Vogue cover shoot" anchors the model better than "beautiful professional lighting"

5. **Specify quantities**: "Two coffee cups" not "coffee cups". "A single person" not "a person" (which might generate multiple)

#### Common Hallucination Triggers & Fixes

| Trigger | Problem | Fix |
|---------|---------|-----|
| "A person holding a product" | Distorted hands/fingers | "Close-up of product on table, hands NOT in frame" or use image-to-video with a real photo |
| "Text on the product label" | Garbled text | Generate image without text, add text in post-production |
| "Multiple people talking" | Face merging | One person per scene, composite in editing |
| "Brand logo visible" | Distorted logo | Add logo as overlay in post, not in prompt |
| "Complex physical interaction" | Physics breaks | Break into simpler shots, edit together |
| "Specific celebrity resemblance" | Legal/ethical issues + poor results | Use descriptive attributes instead |

### 1.3 Camera Movement Vocabulary

Use these precise terms — AI models understand film terminology better than casual descriptions.

| Movement | Description | Best For |
|----------|-------------|----------|
| Push-in | Camera moves toward subject | Building tension, revealing detail |
| Pull-back / Dolly out | Camera moves away from subject | Reveal shots, establishing context |
| Tracking shot | Camera follows subject laterally | Movement, energy, following action |
| Pan (left/right) | Camera rotates on axis | Scanning a scene, transitions |
| Tilt (up/down) | Camera angles up or down on axis | Revealing height, drama |
| Crane up / Crane down | Camera rises or descends vertically | Establishing shots, reveals |
| Orbit / Arc | Camera circles the subject | 360 product views, drama |
| Dolly zoom / Vertigo | Zoom + dolly create disorienting effect | Dramatic moments (use sparingly) |
| Handheld / Steadicam | Slight natural movement | UGC feel, documentary style |
| Static / Locked-off | No movement | Product shots, clean compositions |
| Slow-motion | Reduced playback speed | Emphasizing action, luxury feel |
| Timelapse | Sped-up footage | Process shots, before/after over time |

#### Camera Angle Vocabulary

| Angle | Effect | Use Case |
|-------|--------|----------|
| Eye-level | Neutral, relatable | Talking heads, product demos |
| Low angle | Powerful, aspirational | Luxury products, hero shots |
| High angle | Overview, diminishing | Establishing, flat-lay product |
| Bird's eye / Top-down | Geometric, clean | Flat-lay, food, organized layouts |
| Dutch angle | Tension, unease | Dramatic ads (rare, use carefully) |
| Over-the-shoulder | Intimate, POV | UGC, unboxing, first-person |

### 1.4 UGC Prompt Formulas

These templates generate authentic-feeling content that doesn't look "AI generated."

#### Unboxing Formula
```
A [age] [gender] sitting [location], opening a [color] package.
[Lighting]: Natural [time of day] light from a nearby window, warm tones.
[Camera]: Medium close-up, slightly shaky handheld, phone camera quality.
[Expression]: Genuine surprise and excitement.
[Duration]: 5 seconds.
[Style]: Realistic, casual, user-generated content aesthetic.
```

#### Product Review Formula
```
A [age] [gender] looking directly at camera, holding up a [product].

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [creatify-ai/ai-ad-prompt-guide](https://github.com/creatify-ai/ai-ad-prompt-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

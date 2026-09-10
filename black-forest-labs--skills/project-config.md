---
trigger: always_on
description: > This document is for AI agents and LLMs to follow when working with
---

# FLUX Best Practices

**Version 1.0.0**  
Black Forest Labs  
January 2026

> **Note:**  
> This document is for AI agents and LLMs to follow when working with  
> FLUX image generation prompting and workflows. Humans may also find it useful,  
> but guidance here is optimized for automation and consistency.  

---

## Abstract

Comprehensive prompting and workflow guide for BFL FLUX image generation models. Covers all FLUX.2 and FLUX.1 models including text-to-image, image-to-image editing, JSON structured prompting, color specification, typography, and multi-reference editing. Each rule includes detailed explanations, examples of effective vs ineffective approaches, and model-specific guidance.

---

## Table of Contents

1. [Core Principles](#1-core-principles) - **CRITICAL**
   - 1.1 [Core FLUX Prompting Principles](#11-core-flux-prompting-principles)
2. [Model Selection](#2-model-selection) - **HIGH**
   - 2.1 [FLUX Model Selection Guide](#21-flux-model-selection-guide)
   - 2.2 [FLUX.1 Model Family](#22-flux1-model-family)
   - 2.3 [FLUX.2 Model Family](#23-flux2-model-family)
3. [Text-to-Image Prompting](#3-text-to-image-prompting) - **HIGH**
   - 3.1 [Text-to-Image (T2I) Prompting](#31-text-to-image-t2i-prompting)
4. [Image-to-Image Editing](#4-image-to-image-editing) - **HIGH**
   - 4.1 [Image-to-Image (I2I) Prompting](#41-image-to-image-i2i-prompting)
5. [JSON Structured Prompting](#5-json-structured-prompting) - **MEDIUM-HIGH**
   - 5.1 [JSON Structured Prompting](#51-json-structured-prompting)
6. [Color Specification](#6-color-specification) - **MEDIUM**
   - 6.1 [Hex Color Prompting](#61-hex-color-prompting)
7. [Typography and Text](#7-typography-and-text) - **MEDIUM**
   - 7.1 [Typography and Text Prompting](#71-typography-and-text-prompting)
8. [Multi-Reference Editing](#8-multi-reference-editing) - **MEDIUM**
   - 8.1 [Multi-Reference Image Editing](#81-multi-reference-image-editing)
9. [Positive Prompt Alternatives](#9-positive-prompt-alternatives) - **MEDIUM**
   - 9.1 [Negative Prompt Alternatives](#91-negative-prompt-alternatives)

---

## 1. Core Principles

**Impact: CRITICAL**

Universal prompting principles that apply to all FLUX models. Master these before diving into specific techniques.

### 1.1 Core FLUX Prompting Principles

**Impact: MEDIUM**

These principles apply to all FLUX models and form the foundation of effective prompting.

FLUX does NOT support negative prompts. Always describe what you WANT, not what you don't want.

Build prompts using this structure for consistent results:

**Wrong Approach:**

```text
a portrait of a woman, no glasses, no hat, no makeup
```

**Correct Approach:**

```text
a portrait of a woman with natural skin, clear face, bare head, visible eyes
```

**Example:**

```text
A young woman with flowing auburn hair (subject)
dancing gracefully in mid-leap (action)
in the style of classical oil painting (style)
in a moonlit garden with roses (context)
soft diffused moonlight with subtle rim lighting (lighting)
medium shot, shallow depth of field (technical)
```

More specific prompts yield dramatically better results.

**Vague: Poor Results**

```text
a cat sitting
```

**Specific: Excellent Results**

```text
A fluffy orange tabby cat with bright green eyes sitting regally on a vintage
velvet armchair, afternoon sunlight streaming through lace curtains, warm
golden hour lighting, shallow depth of field, shot on medium format film
```

Write prompts as descriptive prose rather than keyword lists.

**Keyword Style: Less Effective**

```text
woman, portrait, beautiful, blonde, studio, professional, 8k, detailed
```

**Prose Style: More Effective**

```text
A professional studio portrait of a beautiful blonde woman in her thirties,
captured with soft studio lighting that accentuates her features, rendered
in stunning detail with natural skin texture and subtle catchlights in her eyes
```

Always specify lighting - it has the single greatest impact on image quality.

**Natural Lighting:**

- Golden hour - warm, soft, directional

- Overcast - soft, diffused, even

- Harsh midday - high contrast, strong shadows

- Dappled forest light - specular, organic patterns

**Studio Lighting:**

- Softbox - even, professional

- Rim light - edge definition, separation

- Butterfly lighting - beauty, glamour

- Rembrandt lighting - dramatic, classic portraits

**Atmospheric Lighting:**

- Volumetric fog - depth, mystery

- God rays - dramatic, spiritual

- Neon glow - urban, cyberpunk

- Candlelight - warm, intimate

**Mood-Based Lighting:**

- Dramatic shadows - tension, noir

- High key - bright, airy, clean

- Low key - moody, mysterious

- Chiaroscuro - strong contrast, painterly

FLUX prioritizes elements that appear earlier in the prompt. Front-load important elements.

**Less Effective:**

```text
A forest background with soft lighting where a knight in shining armor stands
```

**More Effective:**

```text
A knight in shining armor stands in a forest, soft dappled lighting filtering
through the canopy
```

Optimal prompt length is typically 30-80 words (FLUX can handle up to 512 tokens).

- Too short: Lacks direction, generic results

- Too long: Can become unfocused

- Sweet spot: Enough detail to guide, not so much it confuses


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [black-forest-labs/skills](https://github.com/black-forest-labs/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

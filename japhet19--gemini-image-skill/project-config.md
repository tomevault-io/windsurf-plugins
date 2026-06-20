---
trigger: always_on
description: Generate images using Gemini CLI with the nanobanana extension. Supports icons, patterns, diagrams, illustrations, UI assets, image editing, and presentation slide decks. Use when user needs visuals, backgrounds, app icons, hero images, patterns, flowcharts, presentation slides, or wants to edit/restore existing images.
---


# Google Image Generation Skill

Generate high-quality images using Gemini CLI with the **nanobanana extension** and **Nano Banana Pro** best practices.

---

## Structured Prompt Schema

For best results with Nano Banana Pro, use this structured schema instead of keyword lists:

| Field | Purpose | Example |
|-------|---------|---------|
| **SUBJECT** | Main focus | "cyberpunk hacker in a hood" |
| **CONTEXT** | Environment, mood | "dimly lit server room, tense atmosphere" |
| **COMPOSITION** | Framing, camera | "medium shot, 3/4 angle, subject centered" |
| **STYLE** | Art style, medium | "photorealistic 3D render, cinematic" |
| **LIGHTING** | Light source, palette | "neon blue backlighting, warm orange accents" |
| **TEXT** | Exact words, placement | "headline 'SECURE' top-center, sans-serif bold" |
| **TECHNICAL** | Resolution, aspect | "16:9 widescreen, high detail" |
| **CONSTRAINTS** | What to avoid | "no watermarks, no distorted hands" |

### Schema Example
```bash
~/.claude/skills/google-image/google-img '/generate "SUBJECT: minimalist chat bubble icon. STYLE: modern flat design, iOS aesthetic. LIGHTING: soft ambient, clean white background. TECHNICAL: square 1:1, crisp at 512px. CONSTRAINTS: no text, single blue accent (#0066FF), no shadows"'
```

---

## Three Modes: Choose Based on Task

### Mode A: Production Assets
**For:** Logos, banners, UI elements, thumbnails, marketing materials

- Use **FULL schema** (all 8 fields)
- Emphasize: exact text, brand colors, clean layout
- Keywords: "consistent design system", "grid-based", "print-safe typography"

```bash
# Production mode example
~/.claude/skills/google-image/google-img '/generate "SUBJECT: PlotCraft logo with quill pen. STYLE: modern flat, vector-style. LIGHTING: none (flat design). TEXT: wordmark PlotCraft below icon, sans-serif. TECHNICAL: square, scalable. CONSTRAINTS: max 3 colors, no gradients, works on light/dark backgrounds"'
```

### Mode B: Concept Art & Creative
**For:** Hero images, moodboards, backgrounds, illustrations, artistic exploration

- Use **ABBREVIATED schema** (Subject + Context + Style + Lighting)
- Emphasize: atmosphere, style fusion, cinematic quality
- Keywords: "rich detail", "depth and atmosphere", "blend [style A] with [style B]"

```bash
# Creative mode example
~/.claude/skills/google-image/google-img '/generate "SUBJECT: lone figure in rain. CONTEXT: cyberpunk megacity, neon-lit streets. STYLE: Blade Runner meets anime, cinematic. LIGHTING: neon reflections, volumetric fog" --count=3'
```

### Mode C: Diagrams & Technical
**For:** Flowcharts, architecture diagrams, infographics, wireframes, educational content

- Use **TECHNICAL schema** (Subject + Composition + Text + Constraints)
- Emphasize: clarity, legibility, proper labels
- Keywords: "clear labels", "high contrast", "no overlapping text", "simple geometric shapes"

```bash
# Diagram mode example
~/.claude/skills/google-image/google-img '/diagram "SUBJECT: user auth flow with OAuth. COMPOSITION: left-to-right, decision diamonds, process rectangles. TEXT: clear labels on each step. CONSTRAINTS: no overlapping lines, standard flowchart conventions"'
```

---

## Iterative Refinement Workflow

**Don't expect perfection on the first try.** Treat image generation as a conversation.

### Step 1: Clarify Vague Requests
If the user's prompt is vague ("cool logo", "nice background"), ask:
- What is the **use case**? (app icon, social media, print, web background)
- What **style**? (minimal, detailed, photorealistic, illustrated, abstract)
- Any **text** to include? (exact wording matters)
- **Aspect ratio** preference? (square, 16:9, portrait)
- **Brand colors** or palette?

### Step 2: Generate with Structured Prompt
Transform the user's request into a schema-based prompt, then generate 2-3 variations:
```bash
~/.claude/skills/google-image/google-img '/generate "[structured prompt]" --count=3'
```

### Step 3: Propose Targeted Follow-ups
After generating, suggest 2-3 specific refinements:
- "Tighter crop on the main subject"
- "More minimal layout, remove background elements"
- "Swap color palette to [specific colors]"
- "Keep everything but change expression to [X]"
- "Remove the [element], keep the rest identical"

### Editing Existing Images
Use conversational language for edits:
```bash
~/.claude/skills/google-image/google-img '/edit image.png "keep the composition but change the sky to sunset colors"'
```

---

## Quick Reference

See **[prompt-templates.md](./prompt-templates.md)** for 50+ ready-to-use templates organized by category.

---

## Prerequisites

```bash
# Install Gemini CLI and nanobanana extension
npm install -g @google/gemini-cli
gemini extensions install https://github.com/gemini-cli-extensions/nanobanana

# Ensure API key is set
export GEMINI_API_KEY=<your-key>
```

## Quick Start (Recommended)

Use the `google-img` wrapper script which handles everything automatically:

```bash
~/.claude/skills/google-image/google-img '/generate "description"'
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [japhet19/gemini-image-skill](https://github.com/japhet19/gemini-image-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

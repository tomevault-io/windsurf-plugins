---
trigger: always_on
description: **make-slide** is a universal skill for AI coding agents that generates single-file HTML presentations with speaker notes. The output is always a standalone `.html` file with all CSS and JS inlined — no build step, no framework, no dependencies beyond font CDNs.
---

# make-slide — AI Presentation Skill

## Overview

**make-slide** is a universal skill for AI coding agents that generates single-file HTML presentations with speaker notes. The output is always a standalone `.html` file with all CSS and JS inlined — no build step, no framework, no dependencies beyond font CDNs.

Any AI coding tool (Claude Code, Gemini CLI, Codex, Cursor, etc.) can read this file and follow the instructions to generate professional presentations.

**Repository**: [github.com/kuneosu/make-slide](https://github.com/kuneosu/make-slide)
**Theme Gallery**: [make-slide.vercel.app](https://make-slide.vercel.app)

---

## When to Use

Activate this skill when the user's request matches any of these patterns:

- "Make a presentation about..."
- "Create slides for..."
- "Build a slide deck..."
- "Turn this into a presentation"
- "Make a talk about..."
- "Create a pitch deck for..."
- "Generate presentation slides..."
- Any request involving: **presentation**, **slides**, **deck**, **talk**, **keynote**, **pitch**, **slideshow**

---

## Input Modes

Determine which mode applies based on the user's input:

### Mode A: Topic Only
The user provides a topic and optionally a duration or audience.
```
"Make a 15-min talk about AI trends"
"Create a presentation on microservices architecture"
```
→ You design the structure, content, and speaker notes from scratch.

### Mode B: Content/Material Provided
The user provides source material (documents, notes, data, articles).
```
"Turn this document into a presentation"
"Create slides based on these meeting notes"
```
→ You organize and distill the material into slides + speaker notes.

### Mode C: Script Provided
The user provides a written speech or speaking script.
```
"Create slides for this speech script"
"Make a visual deck that accompanies this talk"
```
→ You analyze the script's structure and create matching visual slides. The script becomes the speaker notes.

---

## Workflow

Follow these steps in order:

### Step 1: Analyze Input
- Determine the input mode (A, B, or C)
- Estimate the appropriate number of slides:
  - 5-min talk → 8-12 slides
  - 10-min talk → 12-18 slides
  - 15-min talk → 18-25 slides
  - 20-min talk → 25-30 slides
  - 30-min talk → 30-40 slides
- Identify the target audience and tone (technical, business, casual, academic)
- Detect the user's language from the conversation (generate content in that language)

### Step 2: Choose a Theme
Present the user with the theme gallery link for browsing:
> **Browse themes here**: https://make-slide.vercel.app

If the user doesn't choose a theme, recommend one based on context:
- Tech/developer talk → `minimal-dark` or `neon-terminal`
- Business/corporate → `corporate` or `minimal-light`
- Startup pitch → `gradient-pop` or `keynote-apple`
- Data/analytics → `data-focus`
- Education/workshop → `paper` or `playful`
- Design/creative → `magazine`
- Product launch → `keynote-apple`
- Casual/team event → `playful`

### Step 3: Check Image Needs
Ask the user:
> "Would you like me to identify image insertion points in the slides? If yes, I'll mark them in the outline and you can provide image URLs. If no, I'll use styled placeholders."

- **Yes** → Mark image positions in the outline, ask user for image URLs
- **No** → Use CSS placeholders (emoji, SVG icons, CSS shapes) that match the theme

### Step 4: Generate Outline
Create a slide-by-slide outline with:
- Slide number
- Slide type (from Slide Types Reference below)
- Title
- Key points / content summary
- Image placement (if applicable)

Present the outline to the user for confirmation. Wait for approval or modifications before proceeding.

### Step 5: Fetch Theme Reference
Fetch the chosen theme's reference files from GitHub:

1. **reference.html** — The complete example presentation with all slide types, styles, navigation, and interactive features. This is your primary template.
2. **README.md** — Design guidelines including color palette, typography, spacing rules, and design philosophy.

Use the raw GitHub URLs listed in the Theme List section below.

### Step 6: Generate HTML
Using the reference.html as your template:
- Replicate the exact HTML structure, CSS styles, and JS functionality
- Replace the demo content with the user's actual content
- Keep all interactive features (navigation, fullscreen, speaker notes, etc.)
- Match the theme's typography, colors, spacing, and animations
- Ensure all slide types used match the patterns in the reference

### Step 7: Generate Speaker Notes
Add speaker notes as `data-notes` attributes on each slide's `<div>`:
```html
<div class="slide" data-notes="Welcome everyone. Today I'll be talking about...">
```
- Notes should be conversational and natural
- Include timing cues where helpful (e.g., "[PAUSE]", "[2 min]")
- Don't just repeat the slide text — expand on it
- Include transitions between slides (e.g., "Now let's move on to...")

### Step 8: Generate Script (Mode A and B only)
For Mode A and B, also generate a separate `script.md` file containing:
- Full speaking script organized by slide

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kuneosu/make-slide](https://github.com/Kuneosu/make-slide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

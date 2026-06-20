---
trigger: always_on
description: Use when user asks to draw, create, or generate diagrams, flowcharts, charts, architecture diagrams, wireframes, or visual illustrations. TRIGGER on "draw", "diagram", "flowchart", "chart", "pie chart", "bar chart", "line chart", "architecture diagram", "excalidraw", "visual", "wireframe", "comparison table", "画图", "流程图", "图表", "架构图", "示意图", "生成图", "做个图".
---


# Excalidraw Diagram Generator

Generate high-quality Excalidraw diagrams via Python script execution using core/ builders.

## When to Use

- User asks to "draw", "generate diagram", "create flowchart", "make architecture diagram"
- User wants visual illustrations for papers, slides, or documentation
- User mentions "excalidraw" or asks for diagram generation
- User wants bar charts, line charts, pie charts, data visualizations, or comparison tables
- User wants to convert SVG icons to Excalidraw elements
- User wants to save and reuse custom icons

---

## Step 0: Quick Intent Capture

### Clear requests (skip questions)

If the user's request is specific enough (e.g., "画一个 3 步流程图 A->B->C", "draw a CI/CD pipeline with 4 stages"), skip detailed questions and output:

```
Got it: [diagram type] | ~[N] elements | Style: [vivid/clean/sketch or default]
Saving to: [suggested path]
```

Then proceed directly to **HARD GATE**.

### Vague requests (ask 3 questions)

If the user's request is vague (e.g., "画个架构图", "make a diagram"), ask these 3 questions **together in a single message**:

1. **"What diagram?"** — Describe the scenario, audience, and purpose.
2. **"Any reference files?"** — Provide file paths or say "none". Supported: CSV, JSON, images, `.excalidraw` files, screenshots.
3. **"Key elements and relationships?"** — What should the diagram show?

If the user provides reference files, **read them** before continuing.

### Output before proceeding

```
Intent confirmed: [diagram type] | [element count] elements | Style: [style]
```

---

## Step 1: Configuration (Condensed)

**Skip entirely if the user already specified these in their request.** Otherwise ask at most 3 questions:

1. **"Style? (vivid/clean/sketch or describe)"**
2. **"Output format? (.excalidraw or .excalidraw.md)"**
3. **"Save path?"** (suggest based on current project — MUST confirm before generating)

### Style Reference

| Style | Colors | Text Size | Gap | Use When |
|-------|--------|-----------|-----|----------|
| `vivid` (alias: `conference`) | 7-color vibrant palette | 14-22pt | 45px | Impressive, information-rich diagrams |
| `clean` (alias: `journal`) | Grayscale only | 8-14pt | 30px | Clear, distraction-free data flow |
| `sketch` (alias: `ppt`) | Warm multi-color | 14-28pt | 55px | Approachable, expressive diagrams |
| `custom` | User-defined from YAML | — | — | Specific visual requirements |

---

## HARD GATE: Read Style Prompt (MANDATORY — Before Writing Any Code)

STOP. You MUST execute these steps IN ORDER. Skipping any step = immediate failure.

### Steps (ALL required — no exceptions)

1. **Use the Read tool** to read `prompts/[style]-prompt.md` (the file for the user's chosen style).
   - If style is not yet chosen, read `prompts/conference-prompt.md` as default.
   - You MUST make a Read tool call — reading from memory or skipping is NOT acceptable.
2. **Output this confirmation to the user** (not silently):
   ```
   > PROMPT CHECK: [style] style loaded.
   > 1. [specific rule from the file you just read]
   > 2. [specific rule from the file you just read]
   > 3. [specific rule from the file you just read]
   ```
   The rules must be QUOTED from the actual file, not paraphrased from memory.

### Bypass Detection

If no Read tool call appears in the conversation before code generation, the HARD GATE was bypassed.
The user can verify this by checking the tool call history.

### Prompt Rules Summary

#### Conference / Vivid Style

- **Grid alignment**: All elements snap to 20px invisible grid
- **Orthographic arrows**: Only 90-degree angles, no diagonal arrows
- **No wasted space**: Compact layout, minimal padding
- **Colors**: Academic blue `#2B5B84`-`#4A90E2`, coral `#E67E22` only for key findings
- **Font**: Helvetica (`fontFamily: 2`), title 20pt, body 12pt, labels 10pt
- **Shapes**: No border radius, `roughness: 0`, borders 1.5pt solid
- **Arrows**: Straight only, 1.5pt, no curved/elbowed arrows

#### Journal / Clean Style

- **Ultra-compact**: Minimum margins, tight spacing
- **Colorblind-safe**: Okabe-Ito palette (Blue `#0072B2`, Orange `#E69F00`, Green `#009E73`, Red `#D55E00`)
- **Font**: Helvetica/Arial (`fontFamily: 2`), title 16pt, body 10pt, labels 8pt
- **Minimum legible text**: 7pt at print scale
- **Shapes**: All borders 0.5-1pt solid, `roughness: 0`, no rounded corners

#### PPT / Sketch Style

- **Generous spacing**: 50px+ gaps between major elements
- **Vibrant colors**: Blue `#1971c2`, Teal `#0c8599`, Orange `#e8590c`, etc.
- **Font**: Any or Virgil (`fontFamily: 1`) for handwritten feel, title 28pt, body 16pt
- **Shapes**: Rounded corners, `roughness: 1`, borders 2pt
- **Arrows**: 2pt, color by semantic role, labels encouraged

---

## Step 2: Generate Diagram

### 2A. Generation Method

There is ONE method: **Python script via `core/` builders.**

- Use `auto_labeled_rect()`, `connect()`, `below()`, `right_of()` for ALL elements and arrows.
- Do NOT write raw `.excalidraw` JSON by hand.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlanYu04/excalidraw-generator-skill](https://github.com/AlanYu04/excalidraw-generator-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

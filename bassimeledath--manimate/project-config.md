---
trigger: always_on
description: Generate diagram and animation videos from natural language descriptions using Manim. Outputs MP4 (default) or GIF on request.
---


# /manimate — Manim Animation Video Maker

Generate diagram and animation videos from natural language descriptions using Manim. Outputs MP4 video by default; GIF available on request.

## Usage

```
/manimate "explain how binary search works"
/manimate "show the Pythagorean theorem proof"
/manimate "visualize bubble sort step by step"
```

## Pipeline

When the user invokes `/manimate`, execute these 12 steps in order:

---

### Step 1: Parameter Inference

Parse the user's prompt and infer rendering parameters:

| Parameter | Range | Default | How to infer |
|-----------|-------|---------|--------------|
| `scenes` | 1-6 | 2 | Count distinct concepts/steps/phases in the prompt |
| `quality` | l/m/h | h | Default high; use medium for quick drafts |
| `format` | gif/mp4/both | mp4 | Default mp4; use gif or both only if user explicitly requests GIF |
| `style` | educational/minimal/cinematic | educational | Infer from the tone/subject |
| `duration_per_scene` | 5-15s | 8 | Longer for complex concepts, shorter for simple transitions |

Write to `$WORK_DIR/params.json`:

```json
{
  "prompt": "explain how binary search works",
  "scenes": 3,
  "quality": "h",
  "format": "mp4",
  "style": "educational",
  "duration_per_scene": 8
}
```

Write `$WORK_DIR/manim.cfg`:

```ini
[CLI]
quality = high_quality
format = mp4
renderer = cairo
disable_caching = True

[output]
media_dir = media
video_dir = {media_dir}/videos
images_dir = {media_dir}/images
```

> **Why `renderer = cairo`**: Cairo is the safe default for headless/CI environments. It requires no GPU, no display server, and no OpenGL context.

---

### Step 2: Preflight Checks

Verify dependencies and set pipeline-wide capability flags:

```bash
# Required: Python 3.8+
python3 --version 2>/dev/null || { echo "python3 not found"; exit 1; }

# Required: ManimCE
python3 -c "import manim; print(f'manim {manim.__version__}')" 2>/dev/null || {
  echo "manim not found. Install: pip install manim"
  exit 1
}

# Required: ffmpeg
command -v ffmpeg >/dev/null 2>&1 || { echo "ffmpeg not found"; exit 1; }

# Optional: LaTeX + dvisvgm
LATEX_AVAILABLE=false
if command -v latex >/dev/null 2>&1 && command -v dvisvgm >/dev/null 2>&1; then
  LATEX_AVAILABLE=true
  echo "LaTeX + dvisvgm available — MathTex/Tex enabled"
else
  echo "LaTeX or dvisvgm not found. Falling back to Text-only mode."
  echo "  Install: brew install --cask mactex-no-gui (macOS) or apt install texlive-full (Linux)"
fi

# Detect timeout command (used for render timeouts in Step 10)
TIMEOUT_CMD=""
if command -v gtimeout >/dev/null 2>&1; then
  TIMEOUT_CMD="gtimeout"
elif command -v timeout >/dev/null 2>&1; then
  TIMEOUT_CMD="timeout"
else
  echo "Neither timeout nor gtimeout found. Render hangs won't be caught."
fi
```

Create a **run-scoped** working directory to allow concurrent pipeline executions:

```bash
WORK_DIR=".manimate-$(date +%s)-$$"
mkdir -p "$WORK_DIR"/scenes "$WORK_DIR"/assets "$WORK_DIR"/lastframes "$WORK_DIR"/output
echo "Working directory: $WORK_DIR"
```

> **`WORK_DIR` is the pipeline root for this run.** All subsequent steps reference `$WORK_DIR` instead of a hardcoded `.manimate` path. This prevents data loss when multiple `/manimate` invocations run concurrently.

> **Pipeline-wide LATEX_AVAILABLE flag**: When `false`, scene code must NOT use MathTex or Tex — use Text() for all text, including math expressions. Render equations as Unicode or ASCII.

---

### Step 3: Story Decomposition

Break the prompt into scenes. Each scene specifies visual elements, animations, and narrative arc.

**Default to SVG icons for every real-world concept.** For each scene, identify concepts that can be icons and add them to `asset_manifest`. Every video should have at least 2 SVG assets — if the manifest is empty, revisit the decomposition.

Use basic Manim shapes **only** for: array cells, flowchart boxes, graphs/axes, code blocks, containers, math expressions. **Everything else gets an SVG.**

Write `$WORK_DIR/story.json` with a top-level `asset_manifest` and per-scene `svg_assets` referencing manifest keys:

```json
{
  "title": "How Binary Search Works",
  "asset_manifest": {
    "magnifier_icon": {
      "description": "Magnifying glass with circular lens and angled handle",
      "viewbox": "0 0 64 64",
      "primary_color_token": "ACCENT",
      "used_in": [1]
    },
    "checkmark_icon": {
      "description": "Bold checkmark inside a rounded square",
      "viewbox": "0 0 64 64",
      "primary_color_token": "SUCCESS",
      "used_in": [3]
    }
  },
  "scenes": [
    {
      "id": 1,
      "title": "The Problem",
      "description": "Show a sorted array of numbers. Highlight that we need to find a target value.",
      "visual_elements": ["sorted array of boxes with numbers", "target value highlighted"],
      "animations": ["Create array", "Highlight target", "Write question text"],
      "svg_assets": ["magnifier_icon"],
      "scene_class": "TheProblem",
      "duration": 8,
      "template": "basic",
      "text_elements": ["title: 3 words", "description: 12 words"],
      "estimated_reading_pauses": 6.0,
      "continuity_in": null,
      "continuity_out": "Array remains visible, target highlighted"
    }
  ],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bassimeledath/manimate](https://github.com/bassimeledath/manimate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

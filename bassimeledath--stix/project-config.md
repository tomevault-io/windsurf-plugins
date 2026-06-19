---
trigger: always_on
description: Generate stick figure animations from natural language descriptions. Outputs GIF and MP4.
---


# /stix — Stick Figure Animation

Generate stick figure animations from natural language descriptions. Outputs GIF and MP4 files.

## Usage

```
/stix "a cat chasing a mouse across a park"
```

## Pipeline

When the user invokes `/stix`, execute these steps in order:

---

### Step 1: Parameter Inference

Parse the user's prompt and infer these parameters. The user never sees or sets these directly — infer them from the natural language description:

| Parameter | Range | Default | How to infer |
|-----------|-------|---------|--------------|
| `scenes` | 1–6 | 2 | Count distinct moments/locations/emotional beats in the prompt |
| `fps` | 8–24 | 8 | Default for most; use 12 for fast action |
| `format` | gif/mp4/both | both | Default both unless user specifies |
| `mood` | playful/melancholy/energetic/calm | playful | Infer from the emotion in the prompt |

Each scene is always **8 seconds** (the animation system is built around this fixed duration).

Write these to `.stix/params.json`:

```json
{
  "prompt": "a cat chasing a mouse across a park",
  "scenes": 3,
  "fps": 8,
  "format": "both",
  "mood": "playful"
}
```

---

### Step 2: Preflight Checks

Verify dependencies are installed:

```bash
command -v agent-browser >/dev/null 2>&1 || { echo "agent-browser not found"; exit 1; }
command -v ffmpeg >/dev/null 2>&1 || { echo "ffmpeg not found"; exit 1; }

# Detect agent CLI (configurable via STIX_AGENT_CLI env var)
STIX_AGENT_CLI="${STIX_AGENT_CLI:-}"
if [ -z "$STIX_AGENT_CLI" ]; then
  if command -v claude >/dev/null 2>&1; then
    STIX_AGENT_CLI="claude -p --dangerously-skip-permissions"
    STIX_AGENT_ENV_UNSET="CLAUDE_CODE_ENTRYPOINT,CLAUDECODE"
  elif command -v codex >/dev/null 2>&1; then
    STIX_AGENT_CLI="codex --quiet --full-auto"
    STIX_AGENT_ENV_UNSET=""
  else
    echo "No supported agent CLI found. Set STIX_AGENT_CLI env var."
    echo "   Supported: claude, codex. Or set STIX_AGENT_CLI to your agent's pipe command."
    exit 1
  fi
fi
AGENT_BIN=$(echo "$STIX_AGENT_CLI" | awk '{print $1}')
command -v "$AGENT_BIN" >/dev/null 2>&1 || { echo "Agent CLI '$AGENT_BIN' not found"; exit 1; }
```

Create the working directory:

```bash
rm -rf .stix
mkdir -p .stix/scenes .stix/assets/characters .stix/assets/props .stix/qc .stix/frames .stix/output
```

---

### Step 3: Story Decomposition

Break the prompt into scenes. The story includes an **asset manifest** that lists every character, background element, and prop needed across all scenes, plus per-scene **position contracts** with exact coordinates.

Write `.stix/story.json`:

```json
{
  "title": "Cat Chases Mouse",
  "assets": {
    "characters": [
      { "id": "cat", "description": "Orange tabby cat (#e8a040), pointy ears, whiskers, curved tail", "expressions_needed": ["determined", "happy"] },
      { "id": "mouse", "description": "Small grey mouse (#999), round ears, thin tail", "expressions_needed": ["neutral", "surprised"] }
    ],
    "background": {
      "description": "Park setting: green grass, 2 trees, bench on the right, blue sky gradient, sun"
    },
    "props": ["tree", "bench"]
  },
  "scenes": [
    {
      "id": 1,
      "description": "A mouse scurries across a park path, unaware.",
      "characters": {
        "mouse": { "start": [50, 330], "end": [500, 330], "start_expr": "neutral", "end_expr": "neutral" }
      },
      "animation_notes": "Mouse runs left to center, scurry animation cycle",
      "props_used": ["tree", "bench"],
      "timing": { "run": "0-100%" },
      "mood": "playful"
    },
    {
      "id": 2,
      "description": "A cat spots the mouse and launches into a chase.",
      "characters": {
        "cat": { "start": [30, 310], "end": [600, 310], "start_expr": "determined", "end_expr": "determined" },
        "mouse": { "start": [500, 330], "end": [850, 330], "start_expr": "neutral", "end_expr": "surprised" }
      },
      "animation_notes": "Cat enters from left chasing mouse, both running right. Cat gains on mouse.",
      "props_used": ["tree"],
      "timing": { "chase": "0-100%" },
      "mood": "energetic",
      "continuity": "Mouse position matches end of scene 1"
    }
  ]
}
```

**Asset manifest rules:**
- Every character that appears in ANY scene must be listed in `assets.characters`
- One shared background for ALL scenes
- All unique props listed in `assets.props`
- Character descriptions include specific colors for consistency

**Per-scene position contracts:**
- `start` / `end` are `[x, y]` pixel coordinates within the 900x400 viewBox
- `start_expr` / `end_expr` specify which expression variant to show
- `timing` maps animation phases to percentages of the 8s duration
- `continuity` notes link scenes together

---

### Step 4: Asset Generation (Parallel Workers)

Generate reusable SVG assets that will be shared across all scenes. Each asset is a self-contained SVG file.

**Output structure:**
```
.stix/assets/
├── characters/
│   ├── cat.svg
│   └── mouse.svg
├── background.svg
└── props/
    ├── tree.svg
    └── bench.svg
```

**The dispatcher must read and inject library file contents into each worker prompt.** Workers run as independent agent subprocesses and cannot access the skill directory.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bassimeledath/stix](https://github.com/bassimeledath/stix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

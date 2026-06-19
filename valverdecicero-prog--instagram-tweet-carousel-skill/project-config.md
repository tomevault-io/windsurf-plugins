---
trigger: always_on
description: >
---


# Instagram Tweet-style Carousel Generator (X / Twitter Layout)

Generates fully self-contained, swipeable HTML carousels where every slide mimics
an authentic X (Twitter) post — designed to be exported as individual 1080×1350px PNGs for Instagram.

---

## Working Directory Rules

**All file operations MUST use the current working directory (cwd) as the base. Never use hardcoded absolute paths like `C:\Users\marco\`.**

- Detect cwd at the start of every session using `pwd` or `os.getcwd()`
- Save all generated files (HTML, PNGs) relative to cwd
- The `ASSETS/` folder is always `{cwd}/ASSETS/`
- The `.claude/launch.json` is always `{cwd}/.claude/launch.json`
- The export output folder is always `{cwd}/slides/`
- Never scan directories outside of cwd for project files or images

---

## Step 0 — Ask Background Preference First

**Before collecting any other details, always ask:**

> "Você prefere o fundo **claro** (branco, estilo X padrão) ou **escuro** (preto, estilo X dark mode)?"

This choice determines the entire color system and font colors. Do not proceed until the user answers.

---

## Step 1: Collect Brand Details

After background preference is confirmed, ask for:

1. **Display name** — shown in the post header (e.g., "Marco Lang")
2. **X handle** — shown below the name (e.g., @marcolang)
3. **Verified badge** — blue checkmark yes/no (default: yes)
4. **Profile photo** — before asking the user anything about the photo, search in this order:
   1. **`{cwd}/assets/`** — check the project folder first (case-insensitive: `assets/` or `ASSETS/`)
   2. **Skill folder `assets/`** — if not found above, check the skill's own assets folder at `~/.claude/skills/instagram-carousel-tweet/assets/`
   - Use the Glob tool or `ls` to list files in each location
   - If an image file is found (JPG, PNG, WEBP) in either location, use it automatically and tell the user which file was picked — do not ask
   - Only ask the user to provide a photo if no image is found in either location
5. **Content / topic** — what the carousel is about
6. **Idioma dos slides** — default: **Português (BR)** unless specified otherwise
7. **Number of slides** — default: 7

If the user provides a website URL or brand assets, derive name and handle from those.

If the user says "make me a carousel about X" without brand details, ask before generating. Don't assume defaults.

---

## Handling User-Provided Images

**This section applies from the very first HTML generation — not only during export.**

The profile photo will be located inside the `ASSETS/` folder in the working directory. Always:

1. Find the file in `ASSETS/` (e.g., `ASSETS/foto.jpg`, `ASSETS/profile.png`)
2. Check the actual file format with the `file` command — extension may lie
3. Embed as base64 `data:` URI — never use relative paths

### ⚠️ Critical Rules

1. **NEVER use relative paths** (`ASSETS/foto.jpg`) — they break in every browser context except the exact folder the HTML lives in.
2. **NEVER use `background: url(filepath)`** — leads to 1.5MB+ base64 inline strings that crash the browser parser.
3. **ALWAYS embed as base64 `data:` URI** — works in preview, export, and any environment.
4. **ALWAYS generate the HTML via Python** (`Path.write_text()`) — shell heredocs interpolate `$` and backticks, corrupting base64 strings.

### Step-by-step: embed the profile photo

```bash
# 1. Check the actual file format (extension may lie)
file ASSETS/profile.jpg
```

```python
import base64
from pathlib import Path

# 2. Read and encode
img_path = Path("ASSETS/profile.jpg")  # adjust filename
# Use "image/jpeg" if `file` command says JPEG, else "image/png"
mime = "image/jpeg"  # or "image/png"
b64 = base64.b64encode(img_path.read_bytes()).decode()
avatar_uri = f"data:{mime};base64,{b64}"

# 3. Inject into HTML template as a Python variable — never via shell
html = f"""...<img src="{avatar_uri}" style="width:100%;height:100%;object-fit:cover;">..."""

Path("carousel.html").write_text(html, encoding="utf-8")
```

### Common image mistakes to avoid

| Mistake | What goes wrong | Fix |
|---------|----------------|-----|
| `<img src="ASSETS/foto.jpg">` | Broken image in browser | Always use base64 `data:` URI |
| `background: url('data:...')` inline with 1.5MB base64 | Browser parser crash | Use `<img>` tag with `object-fit:cover` |
| Generating HTML via shell heredoc | `$` and backtick chars corrupt base64 | Always use Python `Path.write_text()` |
| Assuming `.png` extension = PNG | Wrong MIME type breaks rendering | Run `file` command first |

---

## Step 2: Color System

Two modes only — no gradients, no brand-derived palettes. The X aesthetic is deliberately minimal.

### Light Mode (fundo claro)

```
SLIDE_BG        = #FFFFFF          // Slide background
TEXT_PRIMARY    = #0F1419          // Display name, tweet body text
TEXT_MUTED      = #536471          // Handle, timestamps, muted labels
DIVIDER         = #EFF3F4          // Separator lines, borders
ICON_COLOR      = #536471          // Engagement icons
X_BLUE          = #1D9BF0          // Verified badge, links, accent
SLIDE_NUM_BG    = rgba(0,0,0,0.06) // Slide counter pill background
SLIDE_NUM_TEXT  = #536471          // Slide counter text
```

### Dark Mode (fundo escuro)

```
SLIDE_BG        = #000000          // Slide background

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [valverdecicero-prog/instagram-tweet-carousel-skill](https://github.com/valverdecicero-prog/instagram-tweet-carousel-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

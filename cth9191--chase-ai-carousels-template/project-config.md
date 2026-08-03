---
trigger: always_on
description: This file auto-loads every session. You (Claude Code) read this to know how to drive the carousel-building pipeline end-to-end. The user typed `claude` and is expecting you to take a one-sentence intent and produce a shippable carousel.
---

# Chase AI Carousels — Claude Code Operator Manual

This file auto-loads every session. You (Claude Code) read this to know how to drive the carousel-building pipeline end-to-end. The user typed `claude` and is expecting you to take a one-sentence intent and produce a shippable carousel.

## What this repo is

A template library + production pipeline for Instagram/TikTok carousels. The user is here because they watched a YouTube video and want to build carousels the same way. Your job is to walk them through it — they may have never used this repo before.

## First-time-in-repo behavior

On the user's first message in a fresh repo:

1. **Preflight check** — verify the environment is set up. Run these in parallel:
   - `node --version` → must be ≥ 18
   - `npx playwright --version` → confirms Playwright installed
   - `higgsfield --version` → confirms Higgsfield CLI installed (also accepts `hf` / `higgs` aliases — but `hf` collides with Hugging Face CLI on many systems, prefer `higgsfield`)
   - `higgsfield auth token` → confirms user is authed (prints token if authed, errors if not — `auth status` does NOT exist)
   - (Optional) `ls ~/.claude/skills/higgsfield-generate/SKILL.md` → if present, the official Higgsfield skills bundle is installed (Marketing Studio, Virality Predictor, Soul ID, etc). Not required for cover gen — we call the CLI directly — but nice for adjacent jobs.
2. **If anything fails**, walk them through fix:
   - Node missing → link to <https://nodejs.org/>
   - Playwright missing → tell them to run `npm run setup`
   - Higgsfield CLI missing → see § Higgsfield install (platform-specific) below
   - Higgsfield CLI not authed → `higgsfield auth login` (opens browser, ~5 sec device login)
   - Higgsfield skills bundle missing → optional; only suggest if user asks about Marketing Studio / Virality Predictor / Soul ID. Install: `npx skills add higgsfield-ai/skills`
3. **If all checks pass**, ask what they want to build. Then drive.

### Higgsfield install (platform-specific)

**macOS / Linux:**
```bash
curl -fsSL https://raw.githubusercontent.com/higgsfield-ai/cli/main/install.sh | sh
# OR
brew install higgsfield-ai/tap/higgsfield
# OR
npm install -g @higgsfield/cli
```

**Windows:** `npm install -g @higgsfield/cli` is broken (postinstall tar can't handle Windows paths). Install manually from GitHub releases:
1. Download `hf_<version>_windows_amd64.tar.gz` from <https://github.com/higgsfield-ai/cli/releases/latest>
2. Extract `hf.exe` to a permanent folder (e.g. `C:\Users\<you>\bin\higgsfield\`)
3. Copy `hf.exe` to `higgsfield.exe` in the same folder (so the `higgsfield` command name works — `hf` collides with Hugging Face CLI)
4. Add the folder to user PATH via PowerShell: `[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", "User") + ";C:\Users\<you>\bin\higgsfield", "User")`
5. Open a NEW terminal, then `higgsfield auth login`

If user already in the repo before (carousels folder exists with previous work), skip the preflight — they're past onboarding.

## When the user asks for a new carousel

Trigger phrases: "make a carousel about X", "build me a carousel for Y", "carousel on Z", "/make-carousel".

### Step 1 — Pick a template

Two templates ship in this repo:

| Template | Use when |
|---|---|
| `01-graph-paper-cream` | Top-N lists, tool roundups, framework comparisons. Cream bg + graph paper grid + terracotta accents. |
| `02-terracotta-zine` | 3-step explainers, process tutorials, concept→example pairs. Full-bleed terracotta texture + zine collage. |

Propose ONE based on user's topic. Give the one-line description. Confirm before proceeding.

If neither fits, ask before forking a new template. New templates are a meaningful undertaking — don't quietly invent one.

### Step 2 — Create the dated folder

```
carousels/YYYY-MM-DD-<slug>/
├── assets/
└── (slides.html copied from template)
```

Use **today's date**, never a past date. Use a short kebab-case slug (e.g. `2026-05-22-claude-code-workflows`).

```bash
mkdir -p carousels/<id>/assets
cp templates/<template>/slides.html carousels/<id>/
```

If template has bundled assets (e.g. `bg-zine.png`), also copy:
```bash
cp templates/<template>/assets/*.png carousels/<id>/assets/
```

### Step 3 — Generate the cover (Higgsfield CLI, direct)

The cover is the **only** AI-gen part. Body slides NEVER use AI gen.

**`gpt_image_2` model params** (locked for our cover use — verified via `higgsfield model get gpt_image_2`):
- `prompt` (string, required) — flatten from our JSON prompt template
- `aspect_ratio` — **3:4** (max Higgsfield supports; we crop to 4:5 after)
- `batch_size` — **4** (always 4 variants minimum, never one-shot)
- `quality` — **high**
- `resolution` — **2k** (cheap + sharp enough; bump to 4k only for hero-launch covers)
- `--image <path>` — optional reference image; CLI auto-uploads local file paths

**Workflow:**

1. **Load the prompt template** from `prompts/cover-<style>.json` matching the template:
   - Template 01 → `prompts/cover-marble-statue.json`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cth9191/chase-ai-carousels-template](https://github.com/cth9191/chase-ai-carousels-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

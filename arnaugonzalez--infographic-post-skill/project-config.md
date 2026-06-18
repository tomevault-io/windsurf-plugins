---
trigger: always_on
description: >
---


# Infographic Skill

Generate publication-ready infographics from codebases, data, or text.
Three rendering modes: AI Image (★★★★★), HTML Template (★★★★☆), Matplotlib (★★★☆☆).

---

## Quick Reference — What Command to Run

| User says | Command |
|-----------|---------|
| "Make an infographic of this project" | `python3 scripts/generate_pretty.py --codebase . --output infographic.png` |
| "Architecture infographic for LinkedIn" | `python3 scripts/generate_pretty.py --codebase . --style modern-dark --output infographic.png` |
| "Compare before vs after" | `python3 scripts/generate_pretty.py --codebase . --infographic-type comparison --output compare.png` |
| "Explain how auth works as infographic" | `python3 scripts/generate_pretty.py --codebase . --infographic-type feature --output feature.png` |
| "Show the deployment pipeline" | `python3 scripts/generate_pretty.py --codebase . --infographic-type process --output process.png` |
| "Cheat sheet of our API" | `python3 scripts/generate_pretty.py --codebase . --infographic-type cheatsheet --output cheatsheet.png` |
| "Illustrated style" | Add `--style illustrated` to any command above |
| "Light mode / clean style" | Add `--style modern-light` to any command above |
| "Make it cheap / budget mode" | Add `--legacy-html` to use HTML template instead of AI image |
| "LinkedIn posts from codebase" | `python3 scripts/generate_posts.py . --language en` |
| "Quality audit" | `python3 oss_audit.py --root .` |
| "Pretty KPI dashboard" | `python3 scripts/generate_pretty.py --text "Revenue $2.4M, NPS 72" --type dashboard --output kpis.png` |

---

## Step 1 — Determine the Infographic Type

Ask the user or infer from context:

| Type | Flag | When to use |
|------|------|-------------|
| `architecture` | `--infographic-type architecture` | Default. System components + data flow. |
| `comparison` | `--infographic-type comparison` | Before/after, v1 vs v2, old vs new. |
| `feature` | `--infographic-type feature` | How a specific feature works step-by-step. |
| `process` | `--infographic-type process` | Pipeline, workflow, sequential stages. |
| `cheatsheet` | `--infographic-type cheatsheet` | Reference card, API endpoints, config options. |

## Step 2 — Determine the Visual Style

| Style | Flag | Look |
|-------|------|------|
| `modern-dark` | `--style modern-dark` | Default. Deep navy, glowing orbs, Stripe/Vercel feel. Best for LinkedIn. |
| `illustrated` | `--style illustrated` | Rich gradient, clouds/atmosphere, Canva premium feel. Maximum visual impact. |
| `modern-light` | `--style modern-light` | Clean white, soft shadows, Apple WWDC style. Best for print/docs. |

## Step 3 — Generate

### From a codebase (most common)

```bash
python3 scripts/generate_pretty.py \
  --codebase /path/to/project \
  --title "My App" \
  --infographic-type architecture \
  --style modern-dark \
  --output infographic.png
```

The pipeline:
1. `read_codebase.py` scans the project (skips node_modules, .git, binaries)
2. `content_structurer.py` sends report to LLM → gets structured JSON (layers, connections)
3. `image_prompt_builder.py` builds a designer-brief prompt with brand icon descriptions
4. Gemini image model generates a 1080×1080px publication-quality PNG (~$0.04)
5. Falls back to HTML template if Gemini unavailable

### From inline data

```bash
# Quick architecture from text
python3 scripts/generate_pretty.py \
  --layers "Frontend:React,Next.js|Backend:FastAPI|Database:PostgreSQL,Redis" \
  --title "My App" --output infographic.png

# KPI dashboard
python3 scripts/generate_pretty.py \
  --text "Revenue $2.4M (+18%), NPS 72, Churn 2.1%" \
  --type dashboard --title "Q3 KPIs" --output kpis.png
```

### LinkedIn posts from codebase

```bash
python3 scripts/generate_posts.py /path/to/project --language en
```

Generates two posts (technical + business angle) → stdout + `linkedin_posts.md`.

---

## Rendering Modes

### 🎨 AI Image (default — best quality)

**Cost:** ~$0.04/image | **Requires:** `INFG_API_KEY` (Google AI Studio)

Gemini image model generates a publication-quality PNG directly. Illustrated backgrounds, large brand logos, data flow arrows. Looks like a Canva/Figma design.

```bash
# Default command — uses AI Image automatically if INFG_API_KEY is set
python3 scripts/generate_pretty.py --codebase . --output infographic.png
```

### 🖥️ HTML Template (fallback — free rendering)

**Cost:** ~$0.001 (LLM structuring only) | **Requires:** `INFG_OPENROUTER_API_KEY`

Jinja2 template with glassmorphism, SVG brand icons (3,414 brands), connection arrows. Playwright screenshots to PNG.

```bash
# Force HTML template mode
python3 scripts/generate_pretty.py --codebase . --legacy-html --output infographic.html
```

### 📊 Matplotlib (fully offline — free)

**Cost:** Free | **Requires:** Nothing

```bash
python3 scripts/parse_context.py --root . --output arch.json
python3 scripts/generate_linkedin_arch.py --config arch.json --output arch.png
```

---

## CLI Reference

```bash
python3 scripts/generate_pretty.py \
  --codebase <dir>                    # Read and analyze a codebase
  --title "Title"                     # Override title
  --infographic-type <type>           # architecture|comparison|feature|process|cheatsheet
  --style <style>                     # modern-dark|modern-light|illustrated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arnaugonzalez/infographic-post-skill](https://github.com/arnaugonzalez/infographic-post-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

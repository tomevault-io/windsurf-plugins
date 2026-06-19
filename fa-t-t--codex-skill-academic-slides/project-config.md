---
trigger: always_on
description: Create academic PPT decks, proofsheet PPTX files, poster-style slides, one-slide system architecture diagrams, and paper-report slides as 16:9 image-native presentations using GPT Image 2 through the local `gpt-image-2-skill` backend, with reference-first handling for real figures from txt/md/docx/tex folders and figure directories. Use when the user asks for PPT, slides, presentation slides, report deck, proofsheet, poster, system architecture diagram, academic presentation, poster-like slide, 
---


# PPT Image

This is the canonical PPT-generation skill. It replaces HTML/Beamer/tcbposter-first slide workflows with an image-native workflow: Codex plans the story, writes a strict visual prompt for every slide, generates each 16:9 slide image through `gpt-image-2-skill`, reviews the raster output, then packages the accepted images into a PPTX.

The backend is the installed `gpt-image-2-skill` CLI, normally at `$CODEX_HOME/skills/gpt-image-2-skill/scripts/gpt_image_2_skill.cjs`. Resolve the current skill directory from the loaded `SKILL.md`; commands below use `$PPT_IMAGE_SKILL_DIR` for this skill and `$GPT_IMAGE_2_SKILL_DIR` for the image backend. Do not use `ppt-design`, Beamer, tcbposter, Gemini, Paperbanana, or python-pptx layout reconstruction as the primary rendering route.

If shell commands are needed, set `CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"`, `PPT_IMAGE_SKILL_DIR="$CODEX_HOME/skills/ppt-image"`, and `GPT_IMAGE_2_SKILL_DIR="$CODEX_HOME/skills/gpt-image-2-skill"` unless the skill is installed under different names.

## Hard Boundary

Image generation is not a truth engine. Never ask GPT Image 2 to invent data, citations, bar heights, equations, or experimental numbers. Extract claims and numbers from source files first, lock them in a content manifest, and make the slide prompt reproduce only that manifest. For quantitative plots, use an existing plot as a reference image or create a plain data figure outside the image model, then ask GPT Image 2 to compose or restyle around it while preserving geometry. For formulas, keep only one short core formula per slide; if exact formula rendering fails after retries, stop and report the failure rather than silently accepting a wrong formula.

The default policy is reference-first. If the input contains real figures, plots, diagrams, screenshots, or paper assets, preserve them unless there is a clear reason to redraw. The image model may create surrounding slide layout, explanatory panels, and conceptual visuals, but exact author-provided figures should be inserted as original pixels whenever the slide's correctness depends on them. Use GPT Image 2 to generate a replacement figure only when no suitable source figure exists or when the user explicitly asks for a reinterpretation.

## Output Contract

For every deck, create a self-contained project-local folder:

```text
slides/<slug>_<YYYYMMDD_HHMMSS>/
  source_text.md
  source_inventory.json
  content_manifest.md
  storyboard.md
  style_bible.md
  deck_manifest.json
  assets/
    reference_images/
    original_assets/
  prompts/
    slide_01.md
    slide_02.md
  images/
    slide_01_base.png
    slide_01.png
    slide_02.png
  composites/
    slide_01_composite.json
  review/
    slide_01_review.md
    deck_review.md
  outputs/
    deck.pptx
    contact_sheet.png
    proofsheet.pptx
```

Keep all generated files in this folder. Do not scatter slide images in `figures/`, `outputs/`, or temporary directories unless the user explicitly asks.

## Inputs

Accept three common input shapes without asking the user to reformat:

```text
1. A plain information document: .txt, .md, or .docx.
2. A single source file: .tex, optionally with neighboring figures.
3. A project folder: TeX/Markdown/Word files plus fig/, figures/, images/, or result assets.
```

For Word documents, extract document text and embedded media. For TeX folders, parse text and `\includegraphics{...}` references, then gather figure assets. For plain text or Markdown with no figures, use the content alone and let the agent generate visual structure from the scientific story.

## Template Selection

Before writing the storyboard, choose exactly one primary template and load only that template file from `$PPT_IMAGE_SKILL_DIR/templates/`.

Use `templates/complete_deck.md` for a full PPTX, report deck, paper presentation, group meeting deck, or conference talk.

Use `templates/proofsheet.md` when the user wants a proofsheet, slide overview, visual QA deck, or compact review PPTX from existing slide images. This mode should package existing images and avoid regenerating content unless the user asks for fixes.

Use `templates/poster.md` for a poster, one-page research summary, graphical abstract poster, or poster-style slide.

Use `templates/architecture_diagram.md` for a single system architecture diagram, method pipeline, software stack, experimental workflow, or one-slide technical schematic.

If the request mixes modes, choose the deliverable the user names as final. For example, a full PPTX may also include a proofsheet as a QA artifact, but the primary template remains `complete_deck.md`. Do not combine template constraints in a way that makes the prompt overdetermined.

## Default Visual Style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FA-T-T/codex-skill-academic-slides](https://github.com/FA-T-T/codex-skill-academic-slides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

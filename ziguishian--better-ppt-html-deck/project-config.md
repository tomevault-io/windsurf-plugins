---
trigger: always_on
description: Create better-looking, professional, editable HTML/CSS/JS slide decks. Use when the user asks to make a better-looking PPT, presentation, deck, slide deck, PowerPoint-style HTML deck, visually polished report, course deck, pitch deck, social carousel, or when they provide a topic/material/reference images and want Codex to plan content, choose a visual direction, generate a mandatory style preview for approval, then build an editable HTML deck with preview, present, PDF/PNG/image-PPTX export.
---


# Better PPT HTML Deck

Use this skill to help users quickly create better-looking PPT-style decks rendered from HTML/CSS/JS. The goal is not to build a general PPT workbench, SaaS design tool, Figma clone, or Canva clone. Editing, image replacement, preview, presenting, and export exist only to support fast creation and revision of a more beautiful, more professional deck.

## Hard Rules

- Do not generate visual direction, image assets, or `style-preview.png` when the user's brief is ambiguous. First ask 10-20 Socratic clarification questions, wait for the user's answers, then proceed.
- Generate a style preview before building the final deck project.
- Do not build the final `deck-project/` until the user explicitly approves `style-preview.png` or clearly says to proceed with that style.
- Render the final deck from HTML/CSS/JS. HTML is the source of truth; PDF, PNG, and PPTX are exports.
- All final decks and exports must be 16:9 unless the user explicitly asks for another aspect ratio. Use a fixed 1600x900 (or proportional) render viewport for screenshots/PNG, 16in x 9in for PDF, and an explicit 13.333333in x 7.5in PPTX layout. Do not insert non-16:9 slide screenshots into PPTX pages or stretch individual images into mismatched frames.
- Every final generation must use a fresh project workspace/output directory. Do not overwrite or keep reusing an existing `deck-project/` for a new generation or regenerated deck. Use a unique slug/timestamp path, and ensure exports, screenshots, dev server, and localStorage identity all point to that new project.
- Do not generate whole slides as static AI images. Image generation is allowed only for local visual assets such as cover hero art, background textures, illustrations, product concepts, mood images, or supporting images.
- For final decks, generate bitmap supporting imagery by default unless the user explicitly refuses image generation or the deck is strictly data-only. Do not ship a polished deck with only SVG placeholders, CSS art, or empty image slots.
- Generated images must be default-loaded into the deck: copy them into `deck-project/public/assets/images`, reference those local files from `src/data/deck.json`, render them on the relevant slides through `EditableImage`, and log them in `image-prompts.md` plus `asset-source-log.md`.
- When the user asks for GPT Image, gpt-image-2, AI-generated visuals, richer bitmap artwork, or a style that benefits from texture/atmosphere, use image generation for suitable local assets instead of falling back to plain SVG placeholders.
- Prefer HTML/CSS/SVG for diagrams, architecture drawings, terminal windows, code blocks, data cards, timelines, comparison tables, UI frames, charts, and layout.
- Keep all primary slide text in `src/data/deck.json`; do not hardcode main content inside components.
- Add a unique `deck.projectId` and `deck.dataVersion` to `src/data/deck.json` for each final build. Autosave must use a versioned storage key derived from these fields, and `loadDeck()` must fall back to the bundled seed deck when stored data belongs to an older project/version. Never use a fixed localStorage key across regenerated decks.
- Make the final deck directly editable: every visible title, subtitle, list item, card label, diagram label, code/prompt snippet, quote, speaker note, and image/logo must be editable or replaceable from Edit Mode.
- Include browser-side one-click PPTX export. Do not make users run a terminal command for the primary PPT export path.
- Final projects must be run by Codex before delivery. Do not hand off a deck project with only "run these commands" instructions; install dependencies when needed, run build/validation/screenshot QA, start or verify a local preview/dev server when the project requires one, and give the user the working URL or directly openable file path.
- Include meaningful visual assets or image slots. A final deck with no replaceable images/visuals is incomplete unless the user explicitly requests text-only slides. For polished decks, include generated bitmap assets by default: at minimum one cover/hero bitmap plus 2-5 supporting bitmap images for an 8-12 slide deck, default-loaded into the slides.
- Default layouts must feel substantial and presentation-ready: avoid slides where content only occupies a small island, avoid repeated corner thumbnails, and make primary content plus primary visual occupy most of the slide.
- Build decks as designed page systems, not title-and-bullets pages. Every final deck should use a mix of large hero compositions, split narrative/image pages, KPI/data pages, timelines, comparison tables, quote/testimonial cards, dashboards, roadmap/process pages, and closing CTA pages as appropriate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ziguishian/better-ppt-html-deck](https://github.com/ziguishian/better-ppt-html-deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: Art direction for any content — reads text, PDF, Word, HTML, PPT, then proposes 2-3 creative directions with photography style, mood, and visual language. After selection, generates AI image prompts and visual briefs section-by-section. Use when the user shares content and needs visual direction, image sourcing, or creative direction for any material.
---


# Art Direct

Turn content into visual direction. Point this at anything — a deck, a document, an essay, a brief, a webpage — and get back a creative direction you can actually execute.

## When to Use

- User shares a file (any format) and needs visuals for it
- Developing visual identity for content before building/designing
- Translating written material into photography/illustration direction
- Creating image prompts for AI generation tools
- Art directing a presentation, document, report, or website
- Reviewing existing visuals against content intent (critique mode)

## Supported Inputs

Read content from whatever the user provides:

| Format | How to read |
|--------|-------------|
| `.txt`, `.md` | Read tool directly |
| `.html` | Read tool, strip tags to extract text + structure |
| `.pdf` | Read tool with pages parameter |
| `.docx` | Extract via `python3 -c "import docx; ..."` or `textutil -convert txt` on macOS |
| `.pptx` | Extract via `python3 -c "from pptx import Presentation; ..."` |
| `.rtf` | `textutil -convert txt` on macOS |
| URL | WebFetch tool |

If a format doesn't extract cleanly, ask the user to paste the text.

## The Workflow

```
INGEST CONTENT → ANALYSE → PROPOSE 2-3 DIRECTIONS → USER SELECTS → VISUAL BRIEF + PROMPTS
```

---

## Stage 1: Content Ingestion & Analysis

Read the full content. Extract:

1. **Structure** — What are the units? (slides, sections, chapters, paragraphs, pages)
2. **Core themes** — The 2-3 big ideas the content is actually about
3. **Narrative arc** — Does it build? Contrast? Layer? List?
4. **Audience** — Who receives this? What do they expect to see?
5. **Tone** — Authoritative? Inspirational? Intimate? Provocative? Technical?
6. **Key moments** — Which sections carry the most weight, demand the strongest visuals?
7. **Existing visual language** — If the content already has images, assess what's working and what isn't

**Output a brief content summary** before proceeding. Keep it tight — this is for alignment, not a book report.

---

## Stage 2: Creative Direction Proposals

**If house style template provided** (`--style <name>`):
- Validate content fits the style
- Note any tensions and how to bridge them
- Skip to Stage 3 with adapted style guide

**If no house style:**

Propose **2-3 distinct visual directions**. Each must be genuinely different — not three shades of the same idea. For each:

```
DIRECTION: [Name — a short handle like "Archival Authority" or "Warm Machinery"]

MOOD
What it feels like: [emotional quality in 2-3 words]
Energy: [calm / dynamic / tense / contemplative / electric]

PHOTOGRAPHY STYLE
Type: [documentary / editorial / conceptual / abstract / archival / illustrative]
Subjects: [what appears in the images]
Lighting: [quality of light]
Color treatment: [warm/cool shift, saturation, film stock reference]
Composition: [framing approach]

REFERENCE TOUCHSTONES
"Think [X] meets [Y]" — cite real publications, campaigns, photographers, or brands

WHAT THIS DIRECTION AVOIDS
[Specific clichés and visual tropes this direction rejects]

WHY THIS FITS THE CONTENT
[1-2 sentences connecting direction to content themes]
```

Present all directions. User picks one (or asks for a hybrid). Lock the choice.

---

## Stage 3: Visual Style Guide

Once direction is selected, output the working style guide:

```
VISUAL STYLE GUIDE: [Content Title]
Direction: [Chosen direction]

PHOTOGRAPHY STYLE
─────────────────
Type: [Documentary / Editorial / Conceptual / Abstract / Archival]
Subjects: [What to feature — specific, not generic]
Composition: [Framing rules]
Lighting: [Light quality]
Color treatment: [Color approach, film stock if relevant]

MOOD & TONE
───────────
Primary emotion: [e.g., quiet confidence]
Supporting emotions: [e.g., warmth, precision]
Energy level: [Calm / Dynamic / Tense / Contemplative]

CONSISTENCY RULES
─────────────────
• [Shared quality all images must have]
• [Human subject guidelines]
• [Color palette anchors — hex codes]
• [Aspect ratio defaults]

CLICHÉ BLACKLIST
────────────────
• [Content-specific images to reject]
• [Generic tropes to avoid]
• [Overused metaphors for these themes]

AI GENERATION DEFAULTS
──────────────────────
Photography suffix: [standard prompt additions for photo-style generation]
Illustration suffix: [standard prompt additions for illustration-style generation]
```

---

## Stage 4: Section-by-Section Execution

Work through the content in its natural units (slides, sections, chapters, key passages). For each:

### Step 1: Interpret the section's job
What must the visual communicate? What's the emotional beat?

### Step 2: Apply the Five-Lens Framework

Generate options through five lenses:

| Lens | What it shows | When to use |
|------|---------------|-------------|
| **Literal** | The thing itself, shot with intention | Content is already specific |
| **Human** | People experiencing or doing it | Need emotional connection |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nraford7/art-direct](https://github.com/nraford7/art-direct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

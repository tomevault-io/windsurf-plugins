---
trigger: always_on
description: Convert presentations (PPTX) into scrollytelling HTML with editorial narrative, visual beats, and motion design
---


# Scrollytelling generator

## Skill structure

```
scrollytelling-generator/
├── references/
│   ├── discover-subagent-prompt.md  # discover — subagent task prompt with placeholders
│   ├── beat-types.md        # explore, storyboard — layout patterns, usage, anti-patterns
│   ├── storyboard-spec.md   # storyboard — BEATS JSON structure, format, examples
│   ├── build-guide.md       # build — constraints, scaffold customization, script usage
│   └── quality-check.md     # build — post-build review checklist
├── templates/               # Copy to project directory before editing
│   ├── storyboard-scaffold.html  # storyboard — empty template with render function
│   └── base-scaffold.html        # build — full shell: CSS, JS, progress bar
├── scripts/                 # Run only — not read into context
│   ├── extract.py           # discover — PPTX → slide images + metadata
│   ├── build.py             # build — storyboard BEATS JSON → final HTML
│   └── .venv/               # Python venv for extract.py
├── setup.sh                 # Install dependencies (run once)
└── SKILL.md
```

### Dependencies

- Python 3.8+
- `python-pptx` in venv at `scripts/.venv/` (`python3 -m venv .venv && .venv/bin/pip install python-pptx`)
- LibreOffice (`brew install --cask libreoffice`)

---

## Workflow summary

Four sequential phases. Each phase produces artifacts the next phase consumes. The summary below defines the goal and outputs; the linked process section has all execution steps.

### Routing (`/scrollytelling-generator`)

1. Identify what the user wants to work on and which presentation file to use
2. Determine project state and suggest the next phase:

   | State | Condition | Next phase |
   |---|---|---|
   | Fresh | No artifacts exist | Discover |
   | Discovered | `discover-analysis.md` exists, no `explore-test.html` | Explore |
   | Explored | `explore-test.html` exists, no `storyboard.html` | Storyboard |
   | Storyboarded | `storyboard.html` exists, no `scrollytelling.html` | Build |
   | Built | `scrollytelling.html` exists | Review / iterate |

   If artifacts exist but the user references a different source file, treat as Fresh.

3. Confirm the source file and direction before proceeding

### 1. Discover (`/scrollytelling-generator discover`)

Produces the narrative analysis that all later phases build on. Slide images are read in a subagent so they don't persist in the parent context.

Outputs: `discover-analysis.md` (with visual reference notes table). Follow the [Discover process](#discover) for all execution steps.

### 2. Explore (`/scrollytelling-generator explore`)

Establishes the visual design system and maps slides to beat types. Three stages: design system, beat mapping, proof-of-concept.

Outputs: `explore-test.html`, confirmed design system and layout vocabulary. Follow the [Explore process](#explore) for all execution steps.

### 3. Storyboard (`/scrollytelling-generator storyboard`)

Plans every beat: layout type, editorial copy, sequence, and rationale.

Outputs: `storyboard.html` with BEATS array, all beats confirmed. Follow the [Storyboard process](#storyboard) for all execution steps.

### 4. Build (`/scrollytelling-generator build`)

Generates production HTML from the approved storyboard using the build script.

Outputs: `scrollytelling.html`. Follow the [Build process](#build) for all execution steps.

---

## Guidelines

These apply across all phases.

### Context cost

Slide images dominate context cost. The subagent boundary in discover exists to contain this. Phases that re-read slide images (explore reads 8-10, storyboard should read 0-3) should do so deliberately.

### File handling

- All generated and edited files go in the **project directory** (where the user's presentation lives)
- Never create, edit, or overwrite files inside the skill directory (`~/.claude/skills/scrollytelling-generator/`)
- Skill files are read-only references and shared templates — copy them to the project directory before modifying

### Interaction patterns

- **Quick prompts.** Use the `AskUserQuestion` tool whenever there's a natural next action or decision point. This gives the user clickable options instead of requiring typed responses. Users can always select "Other" to provide custom input. Examples: after generating a file ("Open in browser?" / "Continue to next phase"), after presenting options ("Option A" / "Option B"), after completing a phase ("Proceed to explore" / "Review changes first").
- **Phase kickoff.** After entering a new phase, present the goal and key outputs, then follow the process section for execution steps.

### Tone and approach

- The storyboard is a collaborative planning tool, not a specification document
- Beat rationales should be concise and opinionated (strong opinions, held loosely)
- When suggesting layouts for slides, explain the reasoning so the user can evaluate the editorial judgment, not just the visual result
- Treat text-only beats as first-class content — they do the heaviest narrative lifting
- The goal is always: make the presentation's narrative *work* as a reading experience

---

## Process

### Discover


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lmanheim/scrollytelling-generator](https://github.com/lmanheim/scrollytelling-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

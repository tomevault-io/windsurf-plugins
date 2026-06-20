---
trigger: always_on
description: Generate beautiful, self-contained HTML pages that visually explain systems, code changes, plans, and data. Use when the user asks for a diagram, architecture overview, diff review, plan review, project recap, comparison table, or any visual explanation of technical concepts. Also use proactively when you are about to render a complex ASCII table (4+ rows or 3+ columns) — present it as a styled HTML page instead.
---


# Visual Explainer

Generate self-contained HTML files for technical diagrams, visualizations, and data tables. Always open the result in the browser. Never fall back to ASCII art when this skill is loaded.

**Proactive table rendering.** When you're about to present tabular data as an ASCII box-drawing table in the terminal (comparisons, audits, feature matrices, status reports, any structured rows/columns), generate an HTML page instead. The threshold: if the table has 4+ rows or 3+ columns, it belongs in the browser. Don't wait for the user to ask — render it as HTML automatically and tell them the file path. You can still include a brief text summary in the chat, but the table itself should be the HTML page.

## OpenClaw Prompt Templates

Use the prompt templates in `./prompts/` as reusable workflows. In OpenClaw, these are file-backed prompt guides (not slash commands).

| Prompt | What it does |
|--------|---------------|
| `generate-web-diagram` | Generate an HTML diagram for any topic |
| `generate-visual-plan` | Generate a visual implementation plan for a feature |
| `generate-slides` | Generate a magazine-quality slide deck |
| `diff-review` | Visual diff review with architecture comparison and code review |
| `plan-review` | Compare a plan against the codebase with risk assessment |
| `project-recap` | Mental model snapshot for context-switching back to a project |
| `fact-check` | Verify accuracy of a document against actual code |
| `share` | Deploy an HTML page to Vercel and get a live URL |

When a request matches one of these workflows, read the corresponding file in `./prompts/` and follow it.

## OpenClaw Setup Notes

- **Required:** A local browser (Safari/Chrome/Edge/etc.) to open generated HTML.
- **Output directory:** `~/.openclaw/workspace/diagrams/`.
- **Optional dependency:** `surf` CLI for AI-generated images.
- **Optional dependency:** `vercel-deploy` helper script for the `share` workflow.

Installation/update flow in OpenClaw:
1. Clone/update upstream source (if needed): `git clone https://github.com/nicobailon/visual-explainer.git /tmp/visual-explainer`
2. Sync `references/`, `templates/`, `scripts/`, and `commands/*.md` → `prompts/*.md`
3. Keep `SKILL.md` and `skill.json` in this folder as the OpenClaw-adapted entrypoints

## Workflow

### 1. Think (5 seconds, not 5 minutes)

Before writing HTML, commit to a direction. Don't default to "dark theme with blue accents" every time.

**Visual is always default.** Even essays, blog posts, and articles get visual treatment — extract structure into cards, diagrams, grids, tables.

Prose patterns (lead paragraphs, pull quotes, callout boxes) are **accent elements** within visual pages, not a separate mode. Use them to highlight key points or provide breathing room, but the page structure remains visual.

For prose accents, see "Prose Page Elements" in `./references/css-patterns.md`. For everything else, use the standard freeform approach with aesthetic directions below.

**Who is looking?** A developer understanding a system? A PM seeing the big picture? A team reviewing a proposal? This shapes information density and visual complexity.

**What type of content?** Architecture, flowchart, sequence, data flow, schema/ER, state machine, mind map, class diagram, C4 architecture, data table, timeline, dashboard, or prose-first page. Each has distinct layout needs and rendering approaches (see Diagram Types below).

**What aesthetic?** Pick one and commit. The constrained aesthetics (Blueprint, Editorial, Paper/ink) are safer — they have specific requirements that prevent generic output. The flexible ones (IDE-inspired) require more discipline.

**Constrained aesthetics (prefer these):**
- Blueprint (technical drawing feel, subtle grid background, deep slate/blue palette, monospace labels, precise borders) — see `websocket-implementation-plan.html` for reference
- Editorial (serif headlines like Instrument Serif or Crimson Pro, generous whitespace, muted earth tones or deep navy + gold)
- Paper/ink (warm cream `#faf7f5` background, terracotta/sage accents, informal feel)
- Monochrome terminal (green/amber on near-black, monospace everything, CRT glow optional)

**Flexible aesthetics (use with caution):**
- IDE-inspired (borrow a real, named color scheme: Dracula, Nord, Catppuccin Mocha/Latte, Solarized Dark/Light, Gruvbox, One Dark, Rosé Pine) — commit to the actual palette, don't approximate
- Data-dense (small type, tight spacing, maximum information, muted colors)

**Explicitly forbidden:**
- Neon dashboard (cyan + magenta + purple on dark) — always produces AI slop
- Gradient mesh (pink/purple/cyan blobs) — too generic
- Any combination of Inter font + violet/indigo accents + gradient text


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keylimesoda/visual-explainer-openclaw](https://github.com/keylimesoda/visual-explainer-openclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

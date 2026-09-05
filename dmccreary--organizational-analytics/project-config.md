---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with 
code in this repository.

## Project Overview

An MkDocs Material-based intelligent textbook teaching organizational analytics using
the labeled property graph databases (Cypher) and AI. The site features 15 chapters, interactive MicroSimulations, and a 200-concept learning graph. 
Mascot: Aria the Analytics Ant (indigo/amber color theme).

- **Live site:** https://dmccreary.github.io/organizational-analytics/
- **Repository:** https://github.com/dmccreary/organizational-analytics
- **License:** CC BY-NC-SA 4.0

### Intelligent Textbook Level

This is a **Level 2.9 intelligent textbook** — it focuses on interactivity (MicroSims, learning graph) but does not store student records for hyperpersonalization. Classification follows the five-level framework described in *A Five-Level Classification Framework for Intelligent Textbooks: Lessons from Autonomous Vehicle Standards* (McCreary, December 2025, DOI: 10.35542/osf.io/sh2yu_v1, CC BY-NC-ND 4.0). Paper source: https://github.com/dmccreary/intelligent-textbooks/tree/main/papers/five-levels

### Content Generation

This site was primarily generated from `docs/course-description.md` using Claude Code skills located at https://github.com/dmccreary/claude-skills/tree/main/skills. Most of the work building this site involved adjusting the content generation rules and improving the UI of the MicroSims.

## Development Commands

```bash
# Serve locally (user runs this themselves - never start/stop mkdocs serve)
mkdocs serve

# Build the site
mkdocs build

# Deploy to GitHub Pages
mkdocs gh-deploy
```

There is no package.json, Makefile, or test suite. The project has no npm dependencies. 
JavaScript libraries (vis-network, MathJax) are loaded via CDN.

### Learning Graph Utilities

Python scripts in `docs/learning-graph/` for graph validation and analysis:

```bash
# Validate the learning graph JSON against schema
python docs/learning-graph/validate-learning-graph.py

# Analyze graph metrics (connectivity, DAG validation, dependency chains)
python docs/learning-graph/analyze-graph.py

# Convert CSV edge list to vis-network JSON format
python docs/learning-graph/csv-to-json.py
```

These require `jsonschema` (pip install). All other imports are stdlib.

## Architecture

### Content Pipeline

`mkdocs.yml` is the single source of truth for site navigation and configuration. All content lives under `docs/`. The MkDocs `watch` directive monitors both `docs/` and `mkdocs.yml` for live reload.

### MicroSim Structure

Each MicroSim lives in `docs/sims/<sim-name>/` with these files:

| File | Purpose |
|------|---------|
| `index.md` | Documentation page with iframe embed |
| `main.html` | Entry point (loads JS/CSS, CDN libraries) |
| `script.js` | JavaScript implementation |
| `style.css` or `local.css` | Styling |
| `metadata.json` | Dublin Core metadata (optional) |

**iframe embed pattern** used in `index.md`:
```html
<iframe src="./main.html" width="100%" height="620px"
        style="border: 2px solid #303F9F; border-radius: 8px;"
        allow="fullscreen" allowfullscreen></iframe>
```

The main MicroSim types used are:
1. **vis-network** - Graph visualizations using the vis-network library (CDN)
2. **p5.js** - Canvas-based simulations using p5.js (CDN)

### Custom Admonitions

Two custom admonition types are styled in `docs/css/extra.css`:

- **`prompt`** - Has a copy-to-clipboard button (added by `docs/js/extra.js`)
- **`aria`** - Shows Aria mascot icon in upper-right corner, amber left border

### Learning Graph

`docs/learning-graph/` contains a 200-node, 343-edge concept dependency graph in both CSV and vis-network JSON formats. The graph uses 14 taxonomy categories (FOUND, GMOD, GPERF, EVENT, DPIPE, OMOD, ETHIC, GALG, NLPML, INSIT, APPHR, DASH, CAPST, etc.).

### Key Config Details

- **Theme:** Material with `primary: indigo`, `accent: amber`
- **Math:** MathJax 3 configured in `docs/js/mathjax.js`
- **CSS variables:** Aria color theme defined in `docs/css/extra.css` (see `:root` block)
- **Plugins:** search, social, social_override
- **Markdown extensions:** admonition, pymdownx.superfences, pymdownx.arithmatex (generic mode), pymdownx.highlight (with line numbers), md_in_html, attr_list, footnotes

### Navigation Rules

The `mkdocs.yml` nav section must be updated whenever chapters or MicroSims are added. Never add `- navigation.tabs` to the features list.

---

## Tone and Content Guidelines

The voice is **professional yet approachable** — clear, confident, and encouraging without being stuffy. Use contractions naturally, address readers directly with "you" and inclusive "we," and acknowledge that graph databases and organizational analytics can feel intimidating while reassuring students they'll master it. Frame these skills as gaining a new lens that reveals the hidden dynamics inside any organization.

Humor should be **clever with workplace wit and ant colony puns**, sprinkled throughout (1-2 per major section). Use relatable workplace analogies, organizational situations everyone recognizes, and celebrate insights. Never mock the student — affectionately mock the absurdity of trying to answer graph questions with SQL joins instead.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmccreary/organizational-analytics](https://github.com/dmccreary/organizational-analytics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->

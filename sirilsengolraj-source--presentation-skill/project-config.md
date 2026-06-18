---
trigger: always_on
description: Presentation skill for Codex, ChatGPT agents, and OpenAI-style agents that build, edit, verify, or iterate polished PowerPoint `.pptx` presentations, slides, and slide decks from structured `outline.json` or saved workspaces. Use for editable deck generation, presentation design, source-backed assets, optional generated-image slides, alignment QA, overflow/overlap checks, and reusable deck workspaces. Aliases: `powerpoint-deck-builder`, `pptx-skill`, PowerPoint skill, PPTX skill.
---


# Presentation Skill

Repo-native PowerPoint skill for editable, aligned, QA-checked `.pptx`
decks. The model plans the story and visual strategy; scripts handle fragile
rendering, staging, and verification.

Call this skill as `presentation-skill`. Compatibility aliases:
`powerpoint-deck-builder` and `pptx-skill`.

Search aliases: PowerPoint skill, PPTX skill, presentation skill, slide deck
generator, slides generator, deck builder, presentation generator.

## Non-Negotiables

- Do not write ad hoc inline `python-pptx` or `pptxgenjs` deck code. Author
  `outline.json` and run repo scripts.
- Do not reinstall dependencies during deck generation. Missing dependency:
  report it and stop.
- Do not skip QA for a deliverable deck. Use render-free QA when LibreOffice is
  unavailable.
- Fix source files (`outline.json`, plans, assets, renderer code), not mutated
  `.pptx` artifacts.
- For a new topic, scaffold fresh. Do not clone another deck's structure as a
  house style.

## First Files To Read

- `DESIGN.md`: compact design contract, colors, hierarchy, alignment rules,
  generated-image disclosure.
- `references/outline_schema.md`: accepted `outline.json` fields and variants.
- `references/planning_schema.md`: `design_brief.json`,
  `content_plan.json`, and `evidence_plan.json` shape.
- `references/deck_workspace_mode.md`: saved-workspace workflow.
- `references/agent_picker.md`: concise picker guidance for agents and people.
- `references/visual_qa_prompt.md`: visual-review packet and fresh-eyes inspection prompt.
- `references/editing.md`: only when editing an existing deck.
- `references/pptxgenjs.md`: only when editing JS renderer/templates.

## Workflow

### Quick Deck

Use for one-shot 5-10 slide decks.

```bash
node scripts/build_deck_pptxgenjs.js \
  --outline outline.json \
  --output out.pptx \
  --style-preset <preset>

python3 scripts/qa_gate.py \
  --input out.pptx \
  --outdir /tmp/pptx-qa \
  --style-preset <preset> \
  --strict-geometry \
  --skip-render \
  --fail-on-design-warnings
```

### Saved Workspace

Use when the deck will be extended, audited, or rebuilt later.

```bash
python3 scripts/init_deck_workspace.py \
  --workspace decks/my-deck \
  --title "My Deck" \
  --style-preset <preset>

# edit design_brief.json, content_plan.json, evidence_plan.json,
# asset_plan.json, notes.md, outline.json
python3 scripts/build_workspace.py --workspace decks/my-deck --qa --overwrite
```

Add `--visual-review` once the source text is stable and rendered-slide
judgment matters. It creates `build/qa/visual_review/visual_review.md` plus a
contact sheet for fast source-level iteration.

For public or research topics where credible images should be part of the
deck, use source-backed visual planning:

```bash
python3 scripts/build_workspace.py --workspace decks/my-deck \
  --plan-research-assets --allow-network-assets --qa --overwrite
```

This fills a stub `asset_plan.json` with Wikimedia Commons queries, updates
selected slides to use staged `image:<name>` aliases, writes
`assets/attribution.csv`, and lets the renderer append an Image Sources slide.

Workspace source files:

- `design_brief.json`: audience posture, cover concept, structure strategy,
  grid policy, and card/container rules.
- `content_plan.json`: thesis, audience, slide roles, visual strategy.
- `evidence_plan.json`: sourced facts, metrics, claims, chart candidates.
- `asset_plan.json`: source-backed images, charts, icons, optional generated images.
- `outline.json`: final renderable deck structure.
- `notes.md`: data rules, manual design choices, unresolved assumptions.

## Renderer Policy

- Default renderer: `scripts/build_deck_pptxgenjs.js`.
- Python fallback: `scripts/build_deck.py`, selected by `build_workspace.py`
  only for variants that need python-pptx features such as native charts.
- Mermaid diagrams are optional. Use them only when the process itself is the
  slide's evidence; otherwise use a table, split comparison, figure, or concise
  bullets. `scripts/render_mermaid.py` uses `mmdc` when installed and a
  repo-native fallback otherwise; the fallback caps rows at four boxes and
  balances long flows, but agents should still keep process diagrams short.
- Generated imagery is optional and API-key gated. It must be staged through
  `asset_plan.json` and placed on `variant: "generated-image"` slides unless
  the user explicitly asks otherwise.
- Online source-backed images are also optional and network-gated. For public
  topics where credible images would improve the deck, populate
  `asset_plan.json` with Wikimedia Commons queries and run
  `build_workspace.py --allow-network-assets`; cite those assets through the
  generated `assets/attribution.csv`, slide `sources`, or a final image-source
  slide. If `asset_plan.json` is still a stub, prefer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sirilsengolraj-source/presentation-skill](https://github.com/sirilsengolraj-source/presentation-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

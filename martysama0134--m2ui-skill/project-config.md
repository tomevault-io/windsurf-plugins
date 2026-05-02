---
trigger: always_on
description: When the user asks to create, modify, or generate Metin2 client UI code, follow the instructions below. This includes requests like "create a window", "make a UI", "add a button to X", "modify uiXxx.py", or when they provide a screenshot of a UI to replicate.
---

# m2ui — Metin2 UI Code Generator

When the user asks to create, modify, or generate Metin2 client UI code, follow the instructions below. This includes requests like "create a window", "make a UI", "add a button to X", "modify uiXxx.py", or when they provide a screenshot of a UI to replicate.

## Mode Detection

Detect the appropriate mode from the user's input:

1. **Image attached** — analyze the image and replicate it as Metin2 UI code
2. **Symptom report** — prompt contains a visible-bug phrase ("doesn't appear", "click does nothing", "stuck", "leak", "crashes after", "looks broken", "flickers") — even when a `.py` file is also referenced → load `skills/m2ui/reference/failure-atlas.md` FIRST and diagnose via the matching symptom entry, THEN proceed to the relevant mode
3. **References an existing `.py` file** in `uiscript/` or `root/` — modify that file
4. **Text description** — generate new UI code from the description
5. **No clear intent** — ask the user what they want to do

## Reference Documentation

**Always load these (mandatory floor):**

- `skills/m2ui/reference/mental-model.md` — deprogram web/React assumptions, ymir UI engine concepts
- `skills/m2ui/reference/event-binding.md` — callback wrapping matrix

**Conditional load (read only what applies to the task):**

| You're generating... | Also load |
|----------------------|-----------|
| New window from scratch | The matching anchor from `skills/m2ui/reference/anchors/` (see `anchors/README.md`) |
| Modifying an existing window | Skip anchors. Load just the existing files. |
| Specific widget you haven't used recently | `skills/m2ui/reference/widgets.md` — jump to that widget's section |
| Locale-heavy work | `skills/m2ui/reference/locale.md` |
| Calling a C++ Python API not in context | `skills/m2ui/reference/bindings.md` |
| Patterns reminder | `skills/m2ui/reference/patterns.md` (relevant section only) |
| User describes a visible symptom ("X looks broken", "leak after closing") | `skills/m2ui/reference/failure-atlas.md` — load FIRST before anchors |
| Visual style/sizing for a new window matters | `skills/m2ui/reference/visual-conventions.md` — pick archetype + chrome + palette |

> **Symptom-first dispatch:** When the user reports a visible bug rather than asking for new code, load `skills/m2ui/reference/failure-atlas.md` BEFORE any anchor. Diagnose, then (if a fix means new code) load the relevant anchor.

**Anchor selection** for new windows — 2-step decision tree:

**Step 1 — pick ONE primary archetype (the window's chrome):**

| Window type | Anchor |
|-------------|--------|
| Modal yes/no/text dialog | `skills/m2ui/reference/anchors/01-simple-dialog.md` |
| Board chrome + scrolling DYNAMIC list | `skills/m2ui/reference/anchors/02-board-with-list.md` |
| Form: list of radio-buttons + Accept | `skills/m2ui/reference/anchors/03-list-selector.md` |
| Custom 9-slice bordered panel | `skills/m2ui/reference/anchors/04-9slice-panel.md` |
| Inventory-style with tooltips | `skills/m2ui/reference/anchors/06-tooltip-bound.md` |
| Shop / exchange / trade | `skills/m2ui/reference/anchors/07-shop-exchange.md` |
| Inventory / equipment grid | `skills/m2ui/reference/anchors/08-inventory-equipment.md` |
| Options / settings | `skills/m2ui/reference/anchors/09-options-settings.md` |
| Paginated slot grid | `skills/m2ui/reference/anchors/10-paginated-slot-grid.md` |
| Quest / NPC dialog | `skills/m2ui/reference/anchors/11-quest-npc-dialog.md` |
| Storage / warehouse / mall | `skills/m2ui/reference/anchors/12-storage-warehouse.md` |
| Craft / refine / item-enhancement | `skills/m2ui/reference/anchors/13-craft-refine-window.md` |
| Search / filter dialog with results list | `skills/m2ui/reference/anchors/17-search-filter-dialog.md` |
| Mailbox / message inbox | `skills/m2ui/reference/anchors/18-mailbox-two-pane.md` |
| Daily reward grid / check-in | `skills/m2ui/reference/anchors/19-daily-reward-grid.md` |
| Leaderboard / rank table | `skills/m2ui/reference/anchors/20-leaderboard-table.md` |
| Wheel / roulette / gacha | `skills/m2ui/reference/anchors/21-wheel-roulette.md` |
| Multi-page detail browser | `skills/m2ui/reference/anchors/24-page-history-browser.md` |
| Expandable grouped list | `skills/m2ui/reference/anchors/25-expandable-tree-list.md` |

No exact match → pick CLOSEST. Do NOT skip Step 1.

**Step 2 — pick zero or more augmentors (behaviors layered on top):**

| Behavior | Anchor |
|----------|--------|
| Window guarded by `app.ENABLE_*` | `skills/m2ui/reference/anchors/05-feature-gated.md` |
| Slot↔slot or slot↔window drag-drop | `skills/m2ui/reference/anchors/14-drag-and-drop.md` |
| Driven by `net.Send` / `RecvX` packets | `skills/m2ui/reference/anchors/15-network-coupled-flow.md` |
| Multiple panes switched by tabs / radios | `skills/m2ui/reference/anchors/16-tabbed-content.md` |
| Compare-tooltip side-by-side | `skills/m2ui/reference/anchors/22-compare-tooltip.md` |
| Auto-hide chrome on inactivity timer | `skills/m2ui/reference/anchors/23-auto-hide-chrome.md` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martysama0134/m2ui-skill](https://github.com/martysama0134/m2ui-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

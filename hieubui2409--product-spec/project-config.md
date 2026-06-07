---
trigger: always_on
description: Auto-loaded by Claude Code. Operates the **`cleanmatic:product-spec`** skill (PO-facing, non-technical) on behalf of the product owner. Talk in product language: personas, problems, value, scope, acceptance — not code or engineering jargon.
---

# Product Spec — LLM Operating Guide

Auto-loaded by Claude Code. Operates the **`cleanmatic:product-spec`** skill (PO-facing, non-technical) on behalf of the product owner. Talk in product language: personas, problems, value, scope, acceptance — not code or engineering jargon.

## Five Operating Principles

1. **Frontmatter is source-of-truth**: Always parse YAML (`id`, `status`, `scope`, `moscow`, `horizon`, `personas`, `metrics`); never infer structure from headings.
2. **DRY — one home per fact**: Persona labels → `PRODUCT.md`, narratives → `vision.md`, goals → `brd.md` only, features → PRD, acceptance → stories. Cross-reference by ID.
3. **Script vs LLM split**: Scripts (Python) handle graph/struct (parse, detect orphans, validate enums); LLM judges prose quality, alignment, duplication, contradiction. Always run scripts first.
4. **No silent reversals**: Contradiction with `approved` artifact → surface verbatim with Keep / Change+re-approve / Hybrid — never auto-flip.
5. **Never overwrite prose**: On `--update`, flag affected nodes and ask before regenerating.

---

## Parent-Scoped ID Grammar

| Artifact | ID form                | Example          |
|----------|------------------------|------------------|
| BRD goal | `BRD-G<n>`             | `BRD-G1`         |
| PRD      | `PRD-<SLUG>`           | `PRD-AUTH`       |
| Epic     | `PRD-<SLUG>-E<n>`      | `PRD-AUTH-E1`    |
| Story    | `PRD-<SLUG>-E<n>-S<n>` | `PRD-AUTH-E1-S1` |

`<SLUG>`: must begin with an uppercase ASCII letter; remaining characters may be uppercase letters, digits, or hyphens; ≤16 characters total (enforced regex: `^[A-Z][A-Z0-9-]{0,15}$`). Prefer flat slugs. `<n>`: the next free integer
**within that parent**, allocated by `generate_templates.py`. Parent-scoped means globally unique by construction and
lineage-readable — no central counter.

---

## Bilingual Conventions

`lang: en` (default) or `lang: vi`. Frontmatter keys + IDs stay English always (`BRD-G1`, `PRD-AUTH`, `personas`, `metrics`). Prose + visualizations localize per `lang`; Vietnamese is native-reviewed.

---

## Workflow Pointers (load on demand)

For each flag, load the relevant reference from `.claude/skills/product-spec/references/`:

| Flag                                                                               | Reference                                           |
|------------------------------------------------------------------------------------|-----------------------------------------------------|
| `--product`, `--brd`, `--prd`, no-flag init                                        | `workflow-interview.md`                             |
| `--epic`                                                                           | `workflow-interview.md` (+ `interview-epic.md`)     |
| `--story`                                                                          | `workflow-interview.md` (+ `interview-story.md`)    |
| `--validate`, `--strict`, `--approve`, `--summary`                                 | `workflow-validate.md`                              |
| `--decision` (list/record a PO ruling `DEC-<n>`)                                   | `workflow-validate.md`                              |
| `--apply-critique <report>` (critique return-edge → Keep/Change/Defer → `DEC-<n>`) | `workflow-apply-critique.md` (+ `scripts/parse_critique_report.py`) |
| `--status`                                                                         | `workflow-status.md`                                |
| `--auto`                                                                           | `workflow-auto.md`                                  |
| `--discover <path(s)>` (ingest raw upstream text → candidate persona/problem seeds) | `workflow-discover.md` (+ `scripts/ingest_raw_inputs.py`) |
| `--update`                                                                         | `workflow-update.md`                                |
| `--viz` (incl. `board`/`explorer`), `--format`, `--lang`, `--group-by`, `--layers` | `visualization-spec.md` (+ `scripts/visualize.py`)  |
| `--export`, `--layers`, `--depth`, `--compact-mode`                                | `workflow-export.md` (+ `scripts/render_export.py`) |
| `--reflect` (retroactive memory harvest)                                           | `workflow-reflect.md` (+ `scripts/reflect_scan.py`) |
| engagement knobs `interview_rigor` / `action_prompting` (`preferences.py --set`)   | `workflow-interview.md` → *Engagement profile* (+ `scripts/preferences.py`) |
| memory-write reliability (forcing-functions · opt-in Stop hook · `--memory-hook`)  | `memory-enforcement.md` (the single home)           |
| *(every turn, no flag needed)*                                                     | `guardrails-and-boundaries.md` — **load regardless of flag** |

Load only the references relevant to the active flag — except `guardrails-and-boundaries.md`, which applies on every
turn, flagged or not. Don't pre-load the rest.

---

## Conversation Guardrails (every turn)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hieubui2409/product-spec](https://github.com/hieubui2409/product-spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->

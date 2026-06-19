---
trigger: always_on
description: Generate company-specific business documents: board materials, IR / CFO proposals, internal approval documents, Chinese/Japanese bilingual documents, Word/PDF deliverables, and 1920x1080 HTML/PDF slide decks. Runs a Brain -> Router -> Canvas intelligence pipeline (not theme application): infers the real deliverable, rebuilds structure, challenges weak content, locks numbers and language, then renders and verifies real DOCX/PDF/HTML. Two modes: Quick Mode (drop a source file/text -> polished outp
---


# make-doc-skill v4 — Company Document Generator

Generate polished, company-specific business documents from reusable structure plus company brand tokens. The primary deliverables are **formal Word/PDF board materials, IR / CFO proposals, bilingual Chinese/Japanese documents, and 16:9 HTML/PDF proposal decks.**

This is **not** a theme applier. It runs a Brain -> Router -> Canvas intelligence pipeline that infers the real deliverable, rebuilds structure, fixes language and diagrams, applies per-company brand tokens, and then exports and verifies real DOCX/PDF/HTML.

**Use when:** the user wants to create, refine, or reformat a board material, internal approval document, IR / CFO proposal, business explanation, operating policy, bilingual JP/ZH document, or any formal Word/PDF/slide deliverable — especially when the source company has its own logo, colors, language requirements, compliance tone, and output folder conventions.

## Reference Files

Methodology (loaded on demand):
- `docs/PIPELINE.md` — the 8-step audited Full-Mode pipeline (01_company_tokens -> 08_pdf_qa)
- `docs/PROMPT_ENGINEERING.md` — Brain/Router/Canvas role design + prompt meta-principles
- `docs/DESIGN_RULES.md` — DOCX / deck / diagram / token / brand-color hard constraints
- `docs/SIZE_DEPTH_MATRIX.md` — S/M/L/XL depth control

Optional **HTML output mode** (loaded only when the requested deliverable is an HTML visual one-pager or diagram doc, not a formal Word/PDF document):
- `templates.md` — 18 HTML visual template skeletons
- `strategies.md` — 28 content strategies + expert questioning chains
- `components.md` — 16 HTML component templates
- `depth-rules.md` — HTML depth-visualization rules

The HTML machinery is preserved as a capable secondary output. The company-document pipeline below is the primary path.

---

## Core Methodology — Brain → Router → Canvas

The skill is a three-layer pipeline, not a single-shot generator. Even when Brain rewrites heavily, Canvas never dumps "analytical prose" into a template — Router forces the translation into document structure.

| Layer | Role | Responsibility | Input | Output |
|-------|------|----------------|-------|--------|
| **Brain** | Editor + domain analyst | Understand the real deliverable, rebuild structure, challenge weak content, fix language. This is the **Codex Intelligence Layer**. | Source material + company context | Structured, corrected content + section plan |
| **Router** | Document architect | Map deliverable type -> output family (formal DOCX / split bilingual / 16:9 deck / HTML) -> section schema + renderer + template | Brain output | Composition plan + renderer choice |
| **Canvas** | Renderer + typesetter | Apply tokens, layout rules, diagram rules; render DOCX/HTML; export PDF. Makes **no** editorial decisions. | Composition plan + tokens | DOCX/PDF/HTML + QA |

**Core principle: depth comes from the sharpness of judgment, not the number of steps.** A board document is convincing because Brain read the material correctly and rebuilt it — not because Canvas applied a color theme. Tokens are the design memory; Brain judgment is the execution engine.

**Prompt meta-principles** (apply when writing any sub-prompt in this skill):

1. **Role > Instruction** — say "You are the CFO's board-document editor at a TSE-listed company," not "please improve this."
2. **Constraint > Freedom** — explicit prohibitions ("no fabricated figures") beat vague encouragement.
3. **Structure > Prose** — make the model fill a composition table, do not let it free-write the document.
4. **Few-shot > Description** — one correct DOCX/slide example anchors quality better than ten rules.
5. **Segmented > One-shot** — the pipeline exists so each step's prompt is short, focused, and auditable.

## Two Modes — Quick Mode / Full Mode

| Mode | Trigger | Flow |
|------|---------|------|
| **Quick Mode** | User drops a source (existing Word/PDF/text) and wants a fast polished output, no project setup | Detect input -> extract -> infer real deliverable -> apply company tokens (or sensible default) -> Brain light pass -> render -> export PDF -> report path |
| **Full Mode** | Board/IR material, bilingual delivery, or "build the document properly" | 8-step audited pipeline with intermediate files and user checkpoints (see `docs/PIPELINE.md`) |

Quick Mode trades sparring depth for speed, but **still runs numerical-fidelity and language-lock checks and never fabricates**. After a Quick output, offer "upgrade to Full Mode" (keeps the normalized source, adds tokens/sparring/review).

## Codex Intelligence Layer


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonhnd/makedocskill](https://github.com/jasonhnd/makedocskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

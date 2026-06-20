---
trigger: always_on
description: Comprehensive professional translation skill using Agent Teams. Orchestrates parallel terminology verification, translation with layout preservation, quality review, revision, and professional typesetting — all in one workflow. Use this skill whenever translating documents where accuracy and professional output matter — especially documents with specialized terminology, proper nouns, organization names, or technical content. Triggers on: 'translate with quality check', 'accurate translation', 'v
---


# Translation Quality — Multi-Agent Professional Translation

This skill orchestrates a complete professional translation pipeline using Agent Teams. It combines:
- **Translation with structure preservation** (absorbs translation-layout)
- **Terminology verification via web search** (new capability)
- **Independent quality review** (new capability)
- **Revision based on review feedback** (new capability)
- **Professional document typesetting** (absorbs formal-doc-layout)

## Architecture Overview

```
              User: "翻译这个文档"
                      │
            Team Lead (this session)
            TeamCreate ─ estimate terms
                      │
    ┌─────────────────┼──────────────────┐
    │           ≤50 terms │ >50 terms    │
 translator    term-      │ term-fast    reviewer
 (Sonnet)      researcher │ term-deep    (Opus)
    │          (Sonnet)   │ (both Sonnet) (waiting)
 Phase 1A:     Phase 1B: │ Phase 1B:       │
 First-pass    web search │ parallel        │
 translation   verify     │ search          │
    │              │      │   │  │          │
    │              │    Phase 1.5:          │
    │              │    Lead merges →       │
    │    ← terminology-glossary.json →     │
    ├──────────────┼───────────────────────┤
    │           Phase 2: reviewer          │
    │    ← review feedback →               │
 Phase 3: revision                         │
    │     Phase 4: Typesetting → .docx/.pdf
```

## Prerequisites

Agent Teams must be enabled:
- Add `"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"` to `~/.claude/settings.json` under `"env"`, OR
- Export `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in shell profile

---

## Phase 0: Team Lead Preparation

### 0.1 Read and Analyze Source Document

Read the source document completely. Determine:
- Source language and target language
- Document type (conference agenda, contract, report, academic paper, etc.)
- Estimated length (pages/sections)
- Whether specialized terminology is present

**Source format handling (critical for structure preservation):**
- If source is .doc or .docx, convert with `textutil -convert html` (NOT txt).
  Plain text conversion STRIPS all table structure — the translator cannot
  preserve tables it cannot see.
- Save HTML to /tmp/translation-workspace/source.html
- Also copy the original file for reference

**Chunking (if >10 pages):**
1. Estimate pages: count paragraphs in HTML body, ~35 paragraphs ≈ 1 page
2. If ≤10 pages: skip chunking, proceed as single-chunk mode
3. If >10 pages: identify chapter boundaries in HTML — look for:
   - `<h1>`, `<h2>` tags, OR
   - Bold paragraphs with larger font-size (Word's heading style), OR
   - Paragraphs matching "一、" "二、" "三、" etc. (Chinese section numbering)
4. Group sections into chunks of ≤10 pages each. Never split inside a `<table>` block
5. Save chunks to /tmp/translation-workspace/chunks/chunk-{N}.html
6. Record boundaries in /tmp/translation-workspace/chunk-manifest.md

**Structural manifest** — the lead MUST produce before spawning agents:
Scan the source (use .html version) and record:
- Number of tables, each table's column count and row count
- Content that lives INSIDE table cells (especially multi-item cells like
  "10 presentations listed inside a single schedule row")
Save to /tmp/translation-workspace/structure-manifest.md

### 0.2 Create Workspace

```bash
# Create a workspace directory for this translation
mkdir -p /tmp/translation-workspace
```

All agents will read from and write to this workspace.

### 0.3 Define Contracts

**Before spawning any teammate**, define these contracts:

**Contract 1 — Terminology Glossary (term-researcher → translator, reviewer):**
See `references/glossary-schema.md` for full JSON schema and example.
Confidence: high (2+ sources), medium (1 source), low (best judgment).
Categories: organization, person, place, venue, title, technical, product, event

**Contract 2 — Review Feedback (reviewer → translator):**
See `references/review-feedback-schema.md` for format and example.
Issue types by priority: FABRICATION > OMISSION > TERMINOLOGY > ACCURACY > STRUCTURE > REGISTER

**Contract 3 — Workspace Paths:**
```
/tmp/translation-workspace/
├── source.* / source.html          # Source document + HTML conversion
├── structure-manifest.md           # Lead's structural inventory
├── chunks/                         # Only if >10 pages
│   └── chunk-{N}.html
├── chunk-manifest.md               # Chunk boundaries (if chunked)
├── first-pass.md                   # Merged translation (single or concatenated chunks)
├── first-pass-chunk-{N}.md         # Per-chunk output (if chunked)
├── structure-report[-chunk-{N}].md # Structure audit (per chunk if chunked)
├── glossary-fast.json              # Split mode only: names, places
├── glossary-deep.json              # Split mode only: orgs, technical

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [senshinji/claude-translation-skill](https://github.com/senshinji/claude-translation-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

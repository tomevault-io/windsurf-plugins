---
trigger: always_on
description: Interactively revise slides via 3-step interview workflow
---


# agent4ppt

> **Claude Code plugin** — Parse PPTX templates into markdown, generate PPTX from markdown, and interactively revise slides.
> Compatible with **Claude Code**, **OpenClaw**, and **Codex CLI**.

---

## Installation

```bash
claude plugin marketplace add JinwangMok/agent4ppt
claude plugin install agent4ppt
```

The installer runs `scripts/install.sh` which checks Python ≥ 3.10, installs the three required packages (`python-pptx`, `pyyaml`, `markdown-it-py`), and reports the optional LibreOffice dependency for PNG previews.

### Manual dependency installation

```bash
bash scripts/install.sh
# or directly:
pip install "python-pptx>=0.6.21" "pyyaml>=6.0" "markdown-it-py>=3.0"
```

---

## Skills overview

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| [parse-ppt-template](#parse-ppt-template) | `/parse-ppt-template <pptx_file>` | Parse a PPTX template → markdown template |
| [generate-ppt](#generate-ppt) | `/generate-ppt <markdown_file>` | Generate PPTX from a markdown file |
| [revise-ppt](#revise-ppt) | `/revise-ppt <markdown_file>` | Interactively revise slides (3-step workflow) |

**Markdown is the single source of truth.** The recommended workflow is:

```
PPTX template ──[parse-ppt-template]──► markdown template
                                               │
                              (edit content)   ▼
                               markdown file ──[generate-ppt]──► PPTX output
                                               │
                              (need changes)   ▼
                               markdown file ──[revise-ppt]──► revised PPTX
```

---

## Core design principles

### Canonical document model

Markdown is the **canonical document representation**, not a serialisation format. This means:

- All content edits are made in markdown first; the PPTX is always a derived artifact.
- The `template:` field in YAML frontmatter is the **sole authoritative reference** to the source PPTX. There is no second path to the template.
- `raw_text` inside a placeholder is always **derived from** the structured markdown representation and never treated as an independent field.

### Template contract vs. spatial layout

The placeholder annotation encodes two distinct concerns:

| Concern | Annotation | Meaning |
|---------|-----------|---------|
| **Template contract** | `type:TYPE` | What a placeholder *demands* — the kind of content it accepts (`title`, `body`, `picture`, `chart`, `table`, …). This is the editable semantic contract. |
| **Spatial layout** | `pos:(x,y) size:WxH` | Where the placeholder *lives* on the slide. Informational metadata; preserved passthrough, never independently edited. |

### Passthrough semantics

The following slide properties are **preserved passthrough** — they are neither modelled in markdown nor destroyed during PPTX generation:

- Slide transitions and animations
- Speaker notes
- Master slide properties and theme assets
- Custom XML extensions not relating to content placeholders

---

## Bilingual support

All three skills auto-detect language from the `LANG` environment variable (POSIX locale format, e.g. `ko_KR.UTF-8`).

| `LANG` value | Resolved language |
|---|---|
| `ko`, `ko_KR`, `ko_KR.UTF-8`, … | Korean (`ko`) |
| anything else | English (`en`, default) |

Override per-invocation with the `--lang ko|en` flag. The `lang` field in markdown frontmatter takes the highest precedence for `generate-ppt` and `revise-ppt`.

---

## parse-ppt-template

Parse a PPTX template file and generate a markdown template with placeholder annotations. Placeholders are identified by python-pptx `idx`-based indexing and annotated with `<!-- ph:idx type:TYPE pos:(x,y) size:WxH -->` HTML comments. After writing the output file the skill runs an **automated verification step** that confirms structural invariants without manual inspection.

### Interface

```
/parse-ppt-template <pptx_file> [--output <output.md>] [--lang ko|en] [--layouts-only]
```

Executed via: `python skills/parse-ppt-template/parse_template.py`

### Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `pptx_file` | ✅ | — | Path to the source PPTX template file |
| `--output`, `-o` | ❌ | `<input>.md` | Output path for the generated markdown template |
| `--lang`, `-l` | ❌ | `$LANG` or `en` | Language for guide comments in the output (`ko` \| `en`) |
| `--layouts-only` | ❌ | `false` | Generate one section per layout definition instead of per slide. Useful when the template has no slide content yet. |

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `pptx_file` | File path | A valid PPTX file readable by python-pptx. Must exist on disk. |

### Returns / Output artifacts

A markdown file written to `--output` (or `<input>.md` by default).

#### Exit codes

| Code | Meaning |
|------|---------|
| `0` | Success — output written and all verification checks passed |
| `1` | Input / IO error (file not found, missing dependency, unreadable PPTX) |
| `2` | Verification failure — output was written but failed one or more structural invariant checks |

**Success markers** (printed to stdout, machine-checkable):
```
[agent4ppt] Markdown template written → <path>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JinwangMok/agent4ppt](https://github.com/JinwangMok/agent4ppt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

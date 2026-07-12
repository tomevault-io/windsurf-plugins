---
trigger: always_on
description: This file is loaded automatically by AI agents (Claude Code, Copilot, Codex CLI, Cursor, etc.) to understand the CERG cybersecurity operating model repository and work effectively with it.
---

# CERG — Guide for AI Agents

This file is loaded automatically by AI agents (Claude Code, Copilot, Codex CLI, Cursor, etc.) to understand the CERG cybersecurity operating model repository and work effectively with it.

---

## Project Overview

CERG is an open‑source cybersecurity operating model — not a control framework or compliance checklist. It gives security teams the policies, standards, procedures, workforce architecture, and evidence model to run a real program. Three pillars: Engineering, Risk, and Governance.

## Directory Layout

```
governance/         Policies, governance instruments, competency models, risk framework
standards/          Technical security standards (15 docs)
procedures/         Operational procedures (12 docs)
plans/              Regulatory operational packages (7 docs)
templates/          Fill‑in artifacts for routine work
roles/              Workforce architecture:
  JF-001.md         Job Families Overview
  JF-002.md         NICE Workforce Framework Crosswalk
  jf-exec/          Executive leadership JDs
  jf-seceng/        Security engineering JDs
  jf-riskops/       Risk operations JDs
  jf-govcomp/       Governance & compliance JDs
  jf-adjunct/       Adjunct function JDs
machine-readable/   YAML + JSON artifacts derived from markdown source
tools/              Validation and population scripts
examples/           Adoption profiles
```

## Document Anatomy

Every CERG document follows STY‑001 conventions:

### Metadata Table (top of file)

11‑field table in STY‑001 §4 format:

```
| | |
|---|---|
| **Identifier field** | document identifier value |
| **Version** | X.X |
| **Status** | Approved |
| **Classification** | Public / Internal / Confidential |
| **Owner** | Role Name |
| **Parent Policy** | CERG-POL-001 |
| **Review Cycle** | Quarterly / Annual |
| **Frameworks** | NIST CSF 2.0 · NIST 800‑53r5 · … |
| **Regulations** | NERC‑CIP · CMMC L2 · SOX ITGC · … |
| **Environments** | All in‑scope environments |
```

### Section Numbering

- Top‑level: `## N. Section Title`
- Subsection: `### N.M Subsection Title`
- Must be sequential — no gaps. Section renumbering works from HIGHEST number to LOWEST to prevent cascading replacements.

### Links

- Resolved relative to source file's directory
- Same‑dir: `FILENAME.md`
- Parent dir: `../governance/FILENAME.md`
- Grandparent: `../../governance/FILENAME.md`
- `roles/` → `governance/`: `../governance/FILENAME.md`
- `roles/jf-seceng/` → `governance/`: `../../governance/FILENAME.md`

### Cross‑References

Format: link the CERG document ID to the source Markdown file, not just the ID.

### Document Control Section (last section before appendices)

Contains Revision History (a table), Review Triggers, Related Documents.

## Machine‑Readable Index

`machine-readable/cerg-llm-index.json` contains:

- Per‑document metadata (id, title, type, pillar, status, version, owner, repo-relative path, virtual local-corpus line range, token estimate, summary)
- Prefix registry (POL, STD, PRC, GOV, PLN, TMPL, JF, JD meanings)
- Pillar breakdowns
- Document counts

**Load this first** — it gives you the complete local corpus map. If context is tight, load only the top-level counts and the `documents[].id/path/summary` fields you need.

## Validation

### CI Gate (must pass before committing)

```bash
python3 tools/cerg-validate.py
```

This is the authoritative CI check — requires 0 errors. Common error classes:
- `FILE_NOT_IN_CATALOG` — document not registered in CAT‑001 §5
- `ID_NOT_IN_CATALOG` — cross‑reference to an unregistered ID
- `STATUS_MISMATCH` — file status ≠ catalog status
- `LINK_MISSING` — markdown link target doesn't exist on disk
- `DRAFT_VERSION` — status says Approved but version contains "Draft"
- `RESTRICTED_CLASSIFICATION` — Public doc references Internal/Confidential doc

### Integrity Checker (supplementary)

```bash
python3 tools/cerg-integrity-check.py
```

Broader scan — finds metadata issues, catalog drift, orphan files. It is useful for discovery but is not currently a release gate; prefer `cerg-validate.py` for pass/fail decisions. The validator already supports 4-part workforce IDs such as `CERG-GOV-JD-SECENG-001`.

## Git Workflow for Agents

### Branching

- For cross‑cutting changes (renames, restructures, bulk link fixes): create a feature branch from `main`, work there, merge back.
- For single‑document edits: work directly on `main`.

### Committing

- **One commit per file.** Do not batch multiple files into one commit.
- Commit messages: very short, human‑readable. Examples:
  - `add missing Roles section, renumber`
  - `fix version inconsistency`
  - `update metadata table to STY-001 §4 format`
- For mechanical batch fixes (e.g., same regex applied to 20 files): single commit with a message like `bulk fix: unify Ownership field across per-role JDs`

### Pushing

Push immediately after each commit. Do not accumulate local commits.

```bash
git add FILENAME.md
git commit -m "short message"
git push origin main
```

### Git Config


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m0dernz/CERG](https://github.com/m0dernz/CERG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->

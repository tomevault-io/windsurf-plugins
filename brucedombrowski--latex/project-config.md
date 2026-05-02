---
trigger: always_on
description: Instructions for AI agents working with this repository.
---

# AGENTS.md

Instructions for AI agents working with this repository.

## Repository Overview

**LaTeX Toolkit** - LaTeX templates, build scripts, and PDF tools for producing professional documents in secure, compliance-aware environments.

### Philosophy

This toolkit follows the [SpeakUp project](https://github.com/brucedombrowski/SpeakUp) philosophy:
- **Structured systems** - All work performed in version-controlled, reproducible workflows
- **Traceability** - Source-to-output chains via digital signatures with embedded hashes
- **Automation** - Cross-platform build scripts, AI agent workflows, PDF manipulation
- **Auditability** - Formal decision documentation, compliance markings, signing infrastructure

### Capabilities

| Capability | Component | Status |
|------------|-----------|--------|
| **Decision Memorandums** | Documentation-Generation/DecisionMemorandum/ | Production |
| **Decision Documents** | Documentation-Generation/DecisionDocument/ | Production |
| **Slide Decks** | Documentation-Generation/SlideDecks/ | Planned |
| **Meeting Agendas** | Documentation-Generation/MeetingAgenda/ | Production |
| **CUI Cover Sheets** | Compliance-Marking/CUI/ | Production |
| **Requirements/V&V** | Compliance-Marking/Requirements/ | Production |
| **Export Markings** | Compliance-Marking/Export/ | Planned |
| **PDF Merging** | .scripts/merge-pdf.* | Production |
| **Digital Signatures** | Documentation-Generation/DecisionDocument/ | Production |
| **Software Attestations** | Documentation-Generation/Attestations/ | Production |

### Key Features

**Document Generation:**
- Decision Memorandums (single-page) and Program Decision Documents (multi-page)
- Beamer presentation slide decks
- Structured meeting agendas with timed items
- Software attestations documenting versions, checksums, and download URLs

**Compliance & Security:**
- SF901 CUI cover sheets (32 CFR Part 2002 compliant)
- Export control markings (ITAR/EAR) - planned
- Digital signatures with PIV/CAC smart card support
- Self-signed certificates with source hash traceability

**PDF Operations:**
- Merge multiple PDFs with user-defined ordering
- LaTeX-based (pdfpages) - no external dependencies
- Works on airgapped systems

**Cross-Platform:**
- Windows: `.bat` (double-click), `.ps1` (PowerShell)
- macOS/Linux: `.sh` (bash)
- All tools work offline after LaTeX installation

## Target Environment

**Primary:** Airgapped Windows 11 with security hardening
- CIS Windows 11 Enterprise baseline
- DISA STIG Windows 11 baseline
- Microsoft Security Baseline

**Secondary:** macOS/Linux for development

## Components

| Component | Description | AGENTS.md |
|-----------|-------------|-----------|
| [.scripts/](.scripts/) | Build tools, release scripts, PDF merge utilities | — |
| [.assets/](.assets/) | Shared images and logos | — |
| [.bin/](.bin/) | Executable binaries (PdfSigner.exe) | — |
| [.dist/](.dist/) | Build output (tracked for examples) | — |
| [Documentation-Generation/](Documentation-Generation/) | Document templates (decisions, slides, agendas, attestations) | [Documentation-Generation/AGENTS.md](Documentation-Generation/AGENTS.md) |
| [Decisions/](Decisions/) | Formal Decision Memorandums archive | — |
| [Attestations/](Attestations/) | Generated software attestation PDFs | — |
| [Analysis/](Analysis/) | Technical assessments, DRY analysis, mitigation plans | — |
| [Compliance-Marking/](Compliance-Marking/) | CUI cover pages, export markings, security compliance | [Compliance-Marking/AGENTS.md](Compliance-Marking/AGENTS.md) |

## Documentation-Generation

Document templates following the [SpeakUp project](https://github.com/brucedombrowski/SpeakUp) workflow.

### Workflow Model

```
Mobile Ideation (AI agent, text/voice)
    ↓
IDE-Integrated Execution (LaTeX source → PDF)
    ↓
Verification (build, compliance checks)
    ↓
Distributable Output (PDF)
```

### Documentation Structure

All components follow a consistent `templates/` and `examples/` structure:

```
Documentation-Generation/
├── AGENTS.md                 # Documentation-Generation instructions
│
├── DecisionMemorandum/       # Single-page decision memos
│   ├── templates/
│   │   └── decision_memo.tex
│   ├── examples/
│   └── sign.* -> ../DecisionDocument/sign.*  # Symlinks to signing tools
│
├── DecisionDocument/         # Multi-page program decisions
│   ├── AGENTS.md             # Signing-specific instructions
│   ├── README.md
│   ├── templates/
│   │   └── decision_document.tex
│   ├── examples/
│   ├── sign.sh / sign.ps1 / sign.bat
│   └── PdfSigner.exe         # From github.com/brucedombrowski/PDFSigner
│
├── SlideDecks/               # Presentation slide decks (Beamer)
│   ├── templates/
│   │   └── standard_brief.tex
│   └── examples/
│
├── MeetingAgenda/            # Meeting agenda documents
│   ├── templates/
│   │   └── meeting_agenda.tex
│   └── examples/
│       ├── project_kickoff.tex
│       └── requirements_review.tex
│
├── TechnicalReport/          # Technical reports and assessments
│   ├── templates/
│   │   └── technical_report_base.tex
│   └── examples/
│       └── DRY_Assessment_Report.tex
│
└── Attestations/             # Software attestation documents
    ├── templates/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brucedombrowski/LaTeX](https://github.com/brucedombrowski/LaTeX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->

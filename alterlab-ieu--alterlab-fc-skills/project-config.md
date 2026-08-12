---
trigger: always_on
description: This skill should be used when the user asks about "{trigger phrase 1}", "{trigger phrase 2}",
---

# AlterLab FC Skills — Claude Code Project Instructions

> **Project**: AlterLab FC Skills — 72 Claude AI skills for Faculty of Communication students
> **Owner**: AlterLab Creative Technologies Laboratory
> **Repo**: AlterLab-IEU/AlterLab_Fc

---

## Project Overview

This project generates **72 professional Claude AI skills** organized into 6 department packs for communication faculty students worldwide. Each skill transforms Claude into a domain-specific expert assistant tailored to real coursework, creative production, and professional workflows in communication disciplines.

### Department Packs

| Pack | Folder | Students | Skill Count |
|------|--------|----------|-------------|
| **PRA** — Public Relations & Advertising | `skills/pra/` | PR, advertising, marketing communication | 12 |
| **CDM** — Cinema & Digital Media | `skills/cdm/` | Film, animation, digital media production | 12 |
| **NMC** — New Media & Communication | `skills/nmc/` | New media, journalism, multimedia | 12 |
| **GenAI** — Generative AI Production | `skills/genai/` | Higgsfield, ElevenLabs, Suno workflows | 12 |
| **VCD** — Visual Communication Design | `skills/vcd/` | Graphic design, typography, branding, UI | 12 |
| **RMA** — Research Methods & Academic Writing | `skills/rma/` | Research methodology, academic writing, data analysis | 12 |

### Key Principles
- **UNIVERSAL**: No university-specific references. These skills work for ANY communication student globally.
- **ALTERLAB VOICE**: Follow the AlterLab skill format exactly (see template below).
- **PRODUCTION-READY**: Every skill must be immediately usable — not theoretical, not placeholder.
- **COURSE-MAPPED**: Skills align to standard communication faculty curricula worldwide.
- **AGENTIC**: Every skill autonomously researches, creates file-based deliverables, self-reviews, and iterates.

---

## Skill Architecture

### File Structure
```
AlterLab_Fc/
├── CLAUDE.md                    # This file — project instructions
├── README.md                    # Public-facing repo documentation
├── CONTRIBUTING.md              # Contributing guide
├── package.json                 # Project metadata
└── skills/
    ├── pra/                     # Public Relations & Advertising (12 skills)
    │   ├── alterlab-pra-campaign-strategist/
    │   │   └── SKILL.md
    │   └── ...
    ├── cdm/                     # Cinema & Digital Media (12 skills)
    │   ├── alterlab-cdm-screenwriter/
    │   │   └── SKILL.md
    │   └── ...
    ├── nmc/                     # New Media & Communication (12 skills)
    │   ├── alterlab-nmc-podcast-producer/
    │   │   └── SKILL.md
    │   └── ...
    ├── genai/                   # Generative AI Production (12 skills)
    │   ├── alterlab-genai-text-to-image/
    │   │   └── SKILL.md
    │   └── ...
    ├── vcd/                     # Visual Communication Design (12 skills)
    │   ├── alterlab-vcd-brand-identity/
    │   │   └── SKILL.md
    │   └── ...
    └── rma/                     # Research Methods & Academic Writing (12 skills)
        ├── alterlab-rma-literature-reviewer/
        │   └── SKILL.md
        └── ...
```

Each skill lives in its own folder (`alterlab-{dept}-{name}/SKILL.md`) following the AlterLab NEXUS convention.

### Naming Convention
- **Folder**: `alterlab-{dept}-{skill-name}` (lowercase, hyphenated)
- **Frontmatter name**: `"alterlab-{dept}-{skill-name}"`
- **Collection label**: `Part of the AlterLab FC Skills collection ({Department} department).`

---

## AlterLab Skill Format — MANDATORY TEMPLATE

Every SKILL.md MUST follow this exact structure. Do NOT deviate.

```markdown
---
name: "alterlab-{dept}-{skill-name}"
description: >
  This skill should be used when the user asks about "{trigger phrase 1}", "{trigger phrase 2}",
  "{trigger phrase 3}", "act as {role}", "{role} mode",
  or needs expertise in {one-line capability summary}.
  Part of the AlterLab FC Skills collection ({Department} department).
---

# AlterLab FC {Skill Display Name}

You are **{AgentName}**, {one-sentence role description that establishes expertise and personality}.

### 🧠 Your Identity & Memory
- **Role**: {Specific role title}
- **Personality**: {4 adjectives: strategic, creative, etc.}
- **Memory**: You remember {what patterns/frameworks this agent retains}
- **Experience**: You've {credibility statement about depth of expertise}
- **Execution Mode**: {Agentic description — e.g., "Full agentic: research → create → review → iterate autonomously"}

### 🎯 Your Core Mission

#### {Mission Area 1}
- {Capability bullet 1}
- {Capability bullet 2}
- {Capability bullet 3}
- {Capability bullet 4}

#### {Mission Area 2}
- {Capability bullet 1}
- {Capability bullet 2}
- {Capability bullet 3}
- {Capability bullet 4}

#### {Mission Area 3}
- {Capability bullet 1}
- {Capability bullet 2}
- {Capability bullet 3}
- {Capability bullet 4}

### 🚨 Critical Rules You Must Follow

#### {Domain Standards}
- {Non-negotiable rule 1}
- {Non-negotiable rule 2}
- {Non-negotiable rule 3}
- {Non-negotiable rule 4}

### 📋 Your Core Capabilities

#### {Capability Area 1}
- **{Sub-capability}**: {Description}
- **{Sub-capability}**: {Description}
- **{Sub-capability}**: {Description}

#### {Capability Area 2}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlterLab-IEU/AlterLab-FC-Skills](https://github.com/AlterLab-IEU/AlterLab-FC-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

---
trigger: always_on
description: Overall guidelines about the project.
---

# Microsoft AI Decision Tree - Copilot Instructions

## Project Overview

This is a **comprehensive decision framework and reference guide** for navigating Microsoft's AI technology portfolio. The project helps users systematically evaluate and select the right Microsoft AI technology (M365 Copilot, Copilot Studio, Microsoft Foundry, Agent Service, SDKs, etc.) for their specific business requirements.

**Primary Goal:** Enable evidence-based technology selection through a structured learning journey that progresses from foundational concepts to technical mastery.

**Target Audience:** Technical decision-makers, architects, developers, and business stakeholders evaluating Microsoft AI technologies for enterprise use cases.

---

## Technology Stack

- **Static Site Generator:** Jekyll 4.4+ (GitHub Pages)
- **Documentation Format:** Markdown (.md files)
- **Diagrams:** Mermaid 11.12.1 flowcharts (embedded in Markdown, dark theme configured)
- **Navigation:** Jekyll front matter with `nav_order` property (1-12)
- **Theme:** Just the Docs (remote_theme via GitHub Pages)
- **Custom Styling:** SCSS in `_sass/custom/custom.scss` (no Mermaid overrides; palette is owned inline in diagrams)
- **Ruby Gems:** jekyll-seo-tag, jekyll-github-metadata, jekyll-include-cache, webrick
- **Deployment:** GitHub Pages at https://chrismckee1.github.io/microsoft-ai-decision-tree/

---

## Project Structure & Architecture

### Documentation Organization (12 Core Files)

The documentation follows a **progressive learning flow** designed to build knowledge systematically:

1. **README.md** (nav_order: 1) - Landing page with learning paths
2. **docs/capability-model.md** (nav_order: 2) - **Foundation** - Five capability groupings
3. **docs/decision-framework.md** (nav_order: 3) - **Methodology** - Storybook flow & 9 questions
4. **docs/scenarios.md** (nav_order: 4) - **Context** - Real-world use cases
5. **docs/evaluation-criteria.md** (nav_order: 5) - **Assessment** - Complexity, skills, budget, governance
6. **docs/implementation-patterns.md** (nav_order: 6) - **Execution** - Architecture patterns
7. **docs/technologies.md** (nav_order: 7) - **Deep Dive** - Technical specifications
8. **docs/feature-comparison.md** (nav_order: 8) - **Mastery** - Side-by-side matrices
9. **docs/visual-framework.md** (nav_order: 9) - **Application** - 8 Mermaid decision tree diagrams
10. **docs/quick-reference.md** (nav_order: 10) - **Reference** - Fast lookup tables
11. **docs/resources.md** (nav_order: 11) - **Reference** - External links
12. **docs/glossary.md** (nav_order: 12) - **Reference** - Terminology

### Learning Flow Rationale

**Why this order matters:**
- **Quick Reference at position 10**: Prevents "cheating" - users must learn framework before accessing cheat sheet
- **Scenarios at position 4** (not 9): Provides early context to ground abstract concepts
- **Visual Framework at position 9**: Reinforces methodology after content mastery, preventing it from becoming a shortcut
- **Technologies at position 7** (not 2): Delays deep technical details until after framework understanding

**Navigation links** at the bottom of each document guide users through this optimal progression.

### File Naming & Location Conventions

- Main documentation: `docs/*.md`
- Configuration: `_config.yml`, `Gemfile`
- Custom includes: `_includes/*.html`, `_includes/*.js`
- Custom styles: `_sass/custom/custom.scss`
- Images: `images/*.png`, `images/*.svg`
- GitHub metadata: `.github/copilot-instructions.md` (this file)
- Codex guidance: `AGENTS.md` (root level)

---

## Core Methodology & Validation Rules

### The Constitution (Governing Philosophy)

This repository is governed by a set of immutable principles defined in **[`CONSTITUTION.md`](../CONSTITUTION.md)**. This document serves as the "Supreme Law" for the project.

**Agent Instruction:** You MUST review and adhere to `CONSTITUTION.md` when making decisions.

### Article I: Fact vs. Framework Distinction
**"Cite the Specs, Own the Story."**
- **Technical Truths:** Claims about features, limits, pricing, or availability MUST be validated against official Microsoft documentation. Verify accuracy — include links where they genuinely help the reader, but exhaustive inline citation is not required.
- **Conceptual Truths:** Thought leadership, analogies, and mental models (e.g., "The Coin," "The Podcast") DO NOT require citations. They are the lens we use to teach.

### Source Validation Principle (For Technical Specs)

All technology capabilities, features, and status annotations (GA/Preview/Experimental) MUST be verified against official Microsoft documentation sources.

**Always:**
- Verify against Microsoft Learn documentation first
- If Microsoft Learn does not host the relevant product documentation, use the official product documentation site (VS Code, GitHub Docs, Aspire.dev)
- Include source links where they genuinely help the reader navigate further — not for every statement
- Mark Preview/Experimental/GA status explicitly

### Guardrails for Core Files (Constitutional Mandate)

**CRITICAL:** The following files have **Immutable Intent** defined in `CONSTITUTION.md`.

1.  **`docs/capability-model.md` (The "What")**
    * **Purpose:** Defines abstract capabilities and mental models.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Microsoft-AI-Decision-Framework](https://github.com/microsoft/Microsoft-AI-Decision-Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->

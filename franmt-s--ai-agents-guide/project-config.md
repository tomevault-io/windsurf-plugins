---
trigger: always_on
description: This file defines the strict consistency and formatting rules for any AI agent working within this repository, ensuring that the documentation remains uniform, professional, and accurate as sections are added or modified.
---

# Project Context: AI-Agents-Guide

This file defines the strict consistency and formatting rules for any AI agent working within this repository, ensuring that the documentation remains uniform, professional, and accurate as sections are added or modified.

---

## 0. Repository Map

Before making any change, understand the full layout of this repository:

```text
AI-Agents-Guide/
├── AGENTS.md                      <- This file. Rules for all agents.
├── .gitignore
├── src/                           <- All documentation files live here.
│   ├── es/                        <- Spanish documentation.
│   │   ├── ai-learning-guide.md   <- Main guide. Overview tables and concept intro only.
│   │   │                                 No dense technical details, no massive JSONs here.
│   │   ├── concepts/                  <- Core infrastructure (Shared).
│   │   │   ├── mcp.md                 <- Architecture, best practices, and configs.
│   │   │   └── skills.md              <- Skills system (all tools) & SKILL.md.
│   │   ├── tools/                     <- Individual Agent deep-dives.
│   │   │   ├── antigravity.md         <- VSCode integration & workflows.
│   │   │   ├── claude-code.md         <- CLAUDE.md & Subagents.
│   │   │   ├── codex-cli.md           <- config.toml & Headless mode.
│   │   │   ├── cursor.md              <- .mdc rules & MCP Apps.
│   │   │   ├── gemini-cli.md          <- GEMINI.md & Extensions.
│   │   │   └── opencode.md            <- opencode.json & Lazy loading.
│   │   └── attachments/               <- Localized images (Spanish).
│   ├── templates/                 <- Neutral file templates (shared across languages).
│   │   ├── skills/                <- Skill file templates.
│   │   └── agents/
│   │       ├── monolith/
│   │       │   └── AGENTS.md      <- Single-file AGENTS.md template (small/medium projects).
│   │       └── progressive/
│   │           ├── AGENTS.md      <- Root file: minimal + pointers to secondary docs.
│   │           ├── packages/
│   │           │   └── api/
│   │               └── AGENTS.md      <- Package-level AGENTS.md example (monorepo).
│   │           └── docs/          <- Secondary docs loaded on demand (all lowercase).
│   │               ├── typescript.md
│   │               ├── testing.md
│   │               ├── components.md
│   │               ├── styles.md
│   │               ├── architecture.md
│   │               └── tech-stack.md
│   └── brain/                     <- Agent session notes. Do not edit manually.
├── conductor/                     <- PERSONAL ONLY. Workflow notes and product specs.
│                                     Git-ignored. Do not reference or modify in PRs.
└── spec/                          <- PERSONAL ONLY. Style guides and technical specs.
                                      Git-ignored. Do not reference or modify in PRs.
```

> [!IMPORTANT]
> Never add dense technical content to `src/es/ai-learning-guide.md`. That file only contains summary tables and links to specific tool files in `src/es/`. All detailed configurations go in the tool-specific file.

> [!WARNING]
> The `conductor/` and `spec/` folders are **personal working directories** excluded from version control via `.gitignore`. They exist only on the local machine for the owner's reference.
> - You MAY read them to understand context or intentions when explicitly asked.
> - You MUST NOT suggest committing, publishing, or referencing them in any documentation under `src/`.
> - Do NOT treat their content as project requirements unless the owner explicitly confirms so.

---

## 1. Language and Tone Rules

- **Spanish Prose:** All descriptive and narrative content directed at the end-user MUST be in Spanish.
- **English Technical Terms:** Code snippets, terminal commands, file names, configuration keys (JSON/YAML), and industry terms (e.g., *Hooks*, *Skills*, *Headless Mode*, *Subagents*) MUST remain in English.
- **Code Comments:** Comments inside any code block (including configuration examples) MUST be in English.
- **No Emojis:** Never use emojis in code comments or serious technical writing, unless they are specific visual indicators requested for tables (e.g., ✅, ⚠️, ❌).
- **Narrative Hooks for Titles:** Never use sterile, academic titles like "Reasoning Tokens". You MUST use narrative hooks that communicate a practical consequence, value, or danger (e.g., "Reasoning Tokens: Why Expensive Models Are Worth Every Penny").
- **Engaging Technical Explanations:** Avoid dry Wikipedia-style definitions. When explaining complex architectural concepts, always use relatable human analogies (e.g., "Imagine asking an engineer to design an architecture without a notepad..."). The text must be direct, conversational, and make the user feel they have something practical to lose or gain. Never sound like an academic paper.

---

## 2. Visual Format and Architecture

### Callouts (Obsidian Style)

Always use native Obsidian callouts to highlight information:

```markdown
> [!NOTE]
> For general notes or links to related files.

> [!TIP]
> For best practice advice (e.g., resetting OAuth, using symlinks).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FranMT-S/AI-Agents-Guide](https://github.com/FranMT-S/AI-Agents-Guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: This is the canonical project-context file. All AI coding tools (Claude Code, OpenAI Codex,
---

# pencil-dev-skill: Project Context

This is the canonical project-context file. All AI coding tools (Claude Code, OpenAI Codex,
Cursor, etc.) should read this file for project context. Platform-specific files (`CLAUDE.md`)
are thin pointers to this file.

---

## Project Purpose

> **Unofficial community plugin.** This project is not affiliated with or endorsed by the Pencil.dev team. For the Pencil editor, MCP server, and official documentation, visit [pencil.dev](https://pencil.dev).

This repository is a standalone, **platform-agnostic** AI coding skill plugin that teaches
AI coding tools how to work with [pencil.dev](https://pencil.dev) design files (`.pen` format)
via the Pencil MCP server.

**Core artifact:** `skills/pencil-design/SKILL.md`, the platform-agnostic skill content.
**Platform adapters:** `.claude-plugin/plugin.json`, `.cursor-plugin/plugin.json`,
`.codex-plugin/plugin.json`, and `gemini-extension.json` are the minimum files required
by each platform's installer; they exist only so users on those platforms can run a
one-line install command. They are not the substance of the project.

---

## Naming Conventions

Three names appear in this project, each scoped to a different layer:

| Name | Scope | Where it appears |
|------|-------|-----------------|
| `pencil-skill` | GitHub repo name | Repo URL, clone URL |
| `pencil-dev-skill` | Plugin package name | `plugin.json`, marketplace listings |
| `pencil-design` | Skill name | `SKILL.md` frontmatter, skill activation triggers |

This is intentional: the repo is the deliverable, the plugin is the package, and the
skill is the capability the AI invokes.

---

## Repository Structure

```
skills/pencil-design/              # The platform-agnostic core
  SKILL.md                         # The skill: YAML frontmatter + instructions (v1.11.0)
  references/                      # On-demand references loaded by the skill
    mcp-tools.md                   # Cookbook for all 13 MCP tools + composite recipes
    states.md                      # Component states + screen-level fault states + onboarding/settings states
    flows.md                       # Transitions between screens (modal, validation, back-stack, onboarding, settings, search)
    accessibility.md               # ARIA, focus order, APCA, ARIA live regions, keyboard shortcuts, WCAG 2.2
    modern-patterns.md             # Container queries, fluid type, AI-UI, animation timing, command palette, perceived perf
    pencil-cli.md                  # Full Pencil CLI reference + When CLI vs MCP table
    pen-schema.md                  # .pen file JSON schema reference
    batch-design-grammar.md        # batch_design op syntax (I/C/R/U/G/D/M)
    component-anatomy.md           # Reading component structure: slots, descendants paths, state activation
    composition-patterns.md        # Compound components, slot design, variant naming, status workflow
    file-architecture.md           # Cover frame, section regions, hierarchical naming, multi-.pen layouts
    forms.md                       # Submit behaviour, validation, error display, autofill, mobile inputs
    interactions.md                # Keyboard, focus, hit targets, loading timing, destructive actions, URL-as-state
    visual-hierarchy.md            # Six levers, eye-flow patterns, whitespace, density strategy
    layout-patterns.md             # Hero variations, feature sections beyond three-card grid, pricing, dashboards, settings, list-detail, empty pages (cited 2025/2026 exemplars)
    iteration-patterns.md          # Failure-mode rescues (too busy/sparse/generic/un-premium), self-critique gate, reference-image translation, three-iteration limit
    microcopy.md                   # Voice/tone framework, action-specific CTAs, error message anatomy, empty/success/confirmation copy, loading copy, localisation
    mobile-patterns.md             # Safe areas, sheets vs modals, sheet detents, gestures, haptics, tab bars, native conventions per platform
    iconography.md                 # Stroke weight, sizing, semantic icons, accessibility (aria-hidden vs accessible name), family consistency
    performance-design.md          # Network budgets, Core Web Vitals (LCP/CLS/INP), virtualisation, image and font optimisation, theme-color
    industry-patterns.md           # 8 industry families with 15-20 rules each + completeness pressure tests for SaaS / Website / Mobile
    data-viz.md                    # 25-chart selection matrix, colour-blind palettes (Okabe-Ito, ColorBrewer, Viridis), dashboard tiles, anti-patterns
    style-catalogue.md             # 30+ named UI styles (menu) organised by family with mood, when-to-use, anti-pattern, exemplars
    colour-palettes.md             # 40+ palette recipes (menu) tagged by industry/mood; recipes point to Tailwind/Radix/IBM Carbon/Material 3/Apple HIG
    font-pairings.md               # 30+ typography pairings (menu); recipes point to Google Fonts/Vercel/GitHub/commercial foundries
    codex-tools.md                 # OpenAI Codex tool name mappings
  assets/
    design-system/                 # Optional design-system reference templates
      README.md                    # Agent loading guide

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nisus74/pencil-skill](https://github.com/Nisus74/pencil-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->

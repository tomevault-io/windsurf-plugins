---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repo hosts a **Claude Code skill**, not an application. The skill `c4-model` teaches Claude how to produce C4 architecture diagrams (Simon Brown's model) as Mermaid + accompanying Markdown, across five distinct usage modes (design, document-code, document-prose, review, update) plus supporting diagram variants (Landscape, Deployment, Dynamic). Changes are validated manually by reading the skill end-to-end, by walking the test prompt matrix, and by running the automated test suite in `tests/claude-code/`.

The skill is authored in **English**. Keep prose, examples, and section headings in English when editing `SKILL.md` and its bundled files. The frontmatter `description` intentionally keeps a few French trigger phrases (*"modèle C4"*, *"diagramme d'architecture"*) to widen discoverability for bilingual users.

## Layout and how the pieces connect

```text
.
├── README.md                                     # Public-facing landing page (GitHub visitors)
├── LICENSE                                       # MIT
├── CHANGELOG.md                                  # Release history (Keep a Changelog format)
├── CONTRIBUTING.md                               # How to propose changes
├── CODE_OF_CONDUCT.md                            # Contributor Covenant v2.1 (by reference)
├── CLAUDE.md                                     # This file — guidance for Claude Code
├── .claude-plugin/
│   └── marketplace.json                          # Claude Code marketplace catalog and plugin manifest (single source of truth)
├── commands/                                     # Slash commands registered via marketplace.json
│   ├── auto.md                                   # /c4m:auto — auto-detects the mode from the user's message
│   ├── design.md                                 # /c4m:design — Design mode (greenfield)
│   ├── code.md                                   # /c4m:code — Document-code mode (retro-doc from a repo)
│   ├── prose.md                                  # /c4m:prose — Document-prose mode (retro-doc from README/ADR)
│   ├── review.md                                 # /c4m:review — Review mode (critique or explain)
│   └── update.md                                 # /c4m:update — Update mode
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                                # Always-on: lint, JSON validation, shellcheck, link check
│   │   └── release.yml                           # Tag-triggered: auto-publishes GitHub Release from CHANGELOG
│   └── PULL_REQUEST_TEMPLATE.md                  # PR checklist (editorial invariants, validation steps)
├── .markdownlint.json                            # Markdown lint configuration
├── .gitignore
├── tests/
│   ├── test-prompts.md                           # Manual test matrix (human-readable)
│   └── claude-code/                              # Automated test suite
│       ├── README.md
│       ├── run-skill-tests.sh                    # Orchestrator (--verbose, --integration, --test, --timeout)
│       ├── test-helpers.sh                       # Shared helpers: run_claude, assert_contains, etc.
│       ├── test-c4-model.sh                      # Fast test (~2 min, targeted Q&A)
│       ├── test-c4-model-integration.sh          # Integration test (~3-5 min, full Design workflow)
│       └── fixtures/
│           └── taskflow-prompt.md                # Prompt body for the integration test
└── skills/c4-model/
    ├── SKILL.md                                  # Entry point — router + common contract, loaded every time
    ├── level-template.md                         # Markdown template Claude MUST use for each generated level
    ├── mode-design.md                            # Greenfield brainstorm — 5 dialogued phases
    ├── mode-document-code.md                     # Retro-doc from a codebase (delegates heavy scan to Agent/Explore)
    ├── mode-document-prose.md                    # Retro-doc from prose (README, ADR, spec…)
    ├── mode-review.md                            # Critique or explain an existing diagram
    ├── mode-update.md                            # Evolve an existing C4
    ├── supporting-diagrams.md                    # System Landscape, C4Deployment, C4Dynamic
    ├── mermaid-c4-syntax.md                      # Full Mermaid C4 syntax (sourced from mermaid.js.org)
    ├── review-checklist.md                       # Pre-delivery checklist (sourced from c4model.com)
    └── examples/
        ├── 01-context.example.md                 # Filled-out Context deliverable (Internet Banking System)
        └── 02-container.example.md               # Filled-out Container deliverable (same system)
```

The skill's canonical location in this repo is `./skills/<name>/`, matching the convention used by `anthropics/skills` and community marketplaces. The folder name (`c4-model`) must match the `name` field in `SKILL.md`'s frontmatter **and** in `.claude-plugin/plugin.json`.

### Progressive disclosure pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cheriftj/c4-model-skill](https://github.com/cheriftj/c4-model-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->

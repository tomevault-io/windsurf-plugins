---
trigger: always_on
description: An open-source educational project sharing advanced agentic development patterns learned from production experience. The project serves three audiences:
---

# Development Guidelines for Advanced Agentic Dev Patterns

## Project Overview

### About

An open-source educational project sharing advanced agentic development patterns learned from production experience. The project serves three audiences:

1. **Humans reading docs** — Pattern documentation with architecture diagrams, trade-offs, and real-world context
2. **Humans running examples** — Python scripts and Jupyter notebooks demonstrating each pattern
3. **AI agents consuming skills** — Claude Code skills/plugins providing pattern guidance during development

### Repository Structure

```
advanced-agentic-dev-patterns/
├── patterns/                    # Pattern content (source of truth)
│   ├── tools/                   # Tool design, visibility, output patterns
│   ├── context/                 # Compact, offloading, dynamic block, prefix cache patterns
│   ├── runtime/                 # Dependency injection, graph state hook patterns
│   ├── sandbox/                 # Injection, isolation, persistence patterns
│   ├── plugins/                 # Input, output, runtime extension patterns
│   └── storage/                 # Cache, repository patterns
├── docs/                        # MkDocs content
│   ├── en/                      # English docs (mkdocs.en.yml)
│   └── zh/                      # Chinese docs (mkdocs.zh.yml)
├── sources/                     # Raw source materials (gitignored)
├── wikis/                       # LLM-maintained knowledge base
│   ├── sources/                 # Source summary pages
│   ├── concepts/                # Cross-source concept pages
│   ├── entities/                # People, projects, tools
│   ├── index.md                 # Wiki navigation index
│   └── log.md                   # Operation log
├── skills/                      # Claude Code skills/plugins (structure TBD)
├── scripts/                     # Build and utility scripts
│   ├── build_docs.py            # Copy pattern docs → docs/{lang}/patterns/
│   └── tests/                   # Script tests
├── .claude/skills/              # Project-level Claude Code skills
│   ├── llm-wiki/                # Wiki orchestrator
│   ├── llm-wiki-clip/           # Web → local markdown
│   ├── llm-wiki-ingest/         # Source → wiki pages
│   ├── llm-wiki-lint/           # Wiki health check
│   └── llm-wiki-query/          # Wiki search and Q&A
├── .github/workflows/           # CI/CD
│   ├── ci.yml                   # Lint + test + docs build
│   ├── deploy-docs.yml          # GitHub Pages deployment
│   └── publish-plugin.yml       # Plugin publishing (placeholder)
├── mkdocs.en.yml                # MkDocs config (English, primary)
├── mkdocs.zh.yml                # MkDocs config (Chinese, INHERIT)
├── pyproject.toml               # Project metadata and tool config
├── DEVELOPER.md                 # Setup guide, prerequisites, troubleshooting
└── AGENTS.md                    # This file
```

### Pattern Hierarchy

Each pattern topic contains one or more pattern categories. Each category has three subdirectories:

```
patterns/<topic>/<category>/
├── docs/                        # Pattern documentation (source of truth)
│   ├── overview.md              # English
│   └── overview.zh.md           # Chinese translation
├── examples/                    # Runnable demonstrations
│   ├── basic_usage.py
│   └── basic_usage.ipynb
└── tests/                       # CI-runnable tests
    ├── test_basic_usage.py
    └── conftest.py
```

### Pattern Topics

| Topic | Categories | Focus |
|-------|-----------|-------|
| **tools** | design-patterns, visibility-patterns, output-patterns | How agents use and expose tools |
| **context** | compact-patterns, offloading-patterns, dynamic-block-patterns, prefix-cache-patterns | Managing LLM context windows |
| **runtime** | dependency-injection-patterns, graph-state-hook-patterns | Agent runtime architecture |
| **sandbox** | injection-patterns, isolation-patterns, persistence-patterns | Execution environment management |
| **plugins** | input-patterns, output-patterns, runtime-extension-patterns | Extensibility and plugin systems |
| **storage** | cache-patterns, repository-patterns | Data persistence and caching |

## Internationalization (i18n)

### Language Suffix Convention

Pattern docs use a file suffix to indicate language:

| File | Language |
|------|----------|
| `overview.md` | English (default) |
| `overview.zh.md` | Chinese |
| `overview.ja.md` | Japanese (future) |
| `overview.ko.md` | Korean (future) |

### Build Pipeline

`scripts/build_docs.py` copies pattern docs to MkDocs directories by language suffix:

- `*.md` (no suffix) → `docs/en/patterns/...`
- `*.zh.md` → `docs/zh/patterns/...`

The `docs/{en,zh}/patterns/` directories are **generated** (in `.gitignore`). Never edit them directly — edit files in `patterns/*/docs/` instead.

### MkDocs Configuration

- `mkdocs.en.yml` — primary config with all theme, plugin, and extension settings
- `mkdocs.zh.yml` — inherits from `mkdocs.en.yml`, only overrides `site_name`, `site_description`, `docs_dir`, `site_dir`, and `theme.language`

## Development Environment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PanQiWei/advanced-agentic-dev-patterns](https://github.com/PanQiWei/advanced-agentic-dev-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->

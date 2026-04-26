---
trigger: always_on
description: Instructions for AI coding agents working on this project.
---

# Agents

Instructions for AI coding agents working on this project.

## Maintaining this document

**CRITICAL:** When making architectural changes to the codebase, **you MUST update this AGENTS.md file** to reflect those changes.

This includes:

- **Adding new modules** - Update the "Codebase map" tables with new files
- **Changing architecture patterns** - Update the "Architecture patterns" section
- **Adding new constants** - Update the "Key constants and their locations" table
- **Deprecating patterns** - Remove outdated information and add migration notes
- **Refactoring features** - Update file locations and purposes
- **New testing patterns** - Document in the "Testing" section
- **New development workflows** - Add to relevant sections

**Why this matters:** This document is the primary onboarding guide for AI agents. If it's outdated, agents will:

- Use deprecated APIs
- Create files in the wrong locations
- Follow obsolete patterns
- Break existing conventions

**When to update:** Before completing a PR that changes architecture, review this document and update it. Include AGENTS.md changes in your PR.

**User corrections:** If a user corrects your workflow or tool usage, add a short note here documenting the correction so future agents follow it.

- 2026-01-16: Use `pixi run python` instead of `python` for project commands.
- 2026-01-17: Always run tests after adding tests.
- 2026-01-17: Always run tests after making changes.
- 2026-01-20: NEVER deploy directly to Modal environments (`modal deploy`). Let CI handle Modal deployments via GitHub Actions. Direct deploys bypass testing and can break production.
- 2026-01-20: Fix tag removal bug - canvas uses callback properties (`canvas.onTagRemove = handler`), not event emitter. Changed app.js to set `this.canvas.onTagRemove` instead of `.on('tagRemove', ...)`. Prefer event emitter pattern for new code.
- 2026-01-24: Used `--no-verify` flag with git commit after adding pre-commit hooks (tsc + jsdoc). NEVER use this - always let pre-commit hooks run. If hooks fail, fix issues and commit normally.
- 2026-01-24: Removed TypeScript type checking from pre-commit hooks and pixi tasks. Project uses plain JavaScript with JSDoc annotations for documentation only, not strict type checking.
- 2026-01-25: Consolidated `/code` command handling into CodeFeature plugin. All code node operations (`handleCode`, `handleNodeRunCode`, `handleNodeGenerate`, `handleNodeGenerateSubmit`, `gatherCodeGenerationContext`) have been moved from app.js to plugins/code.js. App.js now delegates to CodeFeature via canvas events (`nodeRunCode`, `nodeGenerate`, etc.).
- 2026-03-15: Always use `encoding="utf-8"` when calling `read_text()`. On Windows, the default encoding is cp1252 which causes UnicodeDecodeError when reading UTF-8 files.
- 2026-03-28: LLM backend migration (LiteLLM → llamabot) design lives under `docs/designs/llamabot-backend-proxy/` (LLD + EARS), linked from HLD and `docs/llds/backend-llm-proxy.md`. PocketFlow is not part of that plan.
- 2026-03-28: Pin `llamabot>=0.18.0` in `pyproject.toml` (PyPI includes `AsyncSimpleBot`/`AsyncStructuredBot`). Backend still uses sync bots + `asyncio.to_thread` until migrated.

**Python commands:** Use `pixi run python` when running project Python commands so the pixi environment and dependencies are active.

## Codebase map

Quick reference for which files to edit for common tasks.

### Design documents (LLM backend migration)

| Location | Purpose |
| -------- | ------- |
| [`docs/designs/llamabot-backend-proxy/LLD.md`](docs/designs/llamabot-backend-proxy/LLD.md) | Low-level design: llamabot SimpleBot/StructuredBot adapter; LiteLLM retained for images, token count, Copilot model list |
| [`docs/designs/llamabot-backend-proxy/llm-proxy-EARS.md`](docs/designs/llamabot-backend-proxy/llm-proxy-EARS.md) | EARS requirements for API parity and bot usage |
| [`docs/llds/backend-llm-proxy.md`](docs/llds/backend-llm-proxy.md) | Narrative LLM proxy + links to the LLD |

### Directory structure

```text
canvas-chat/
├── src/canvas_chat/          # Main Python package
│   ├── app.py                # FastAPI backend routes
│   ├── llm_messages.py       # OpenAI-style dicts → llamabot message tuples
│   ├── config.py             # Configuration management
│   ├── __main__.py           # CLI entry point
│   └── static/               # Frontend assets
│       ├── index.html        # Main HTML
│       ├── css/              # Stylesheets (9 files)
│       └── js/               # JavaScript modules
│           ├── app.js        # Main application orchestrator
│           ├── canvas.js    # SVG canvas rendering
│           ├── crdt-graph.js # Graph data model
│           ├── chat.js      # LLM API integration
│           ├── feature-*.js # Feature plugins
│           └── example-plugins/ # Example plugins (test, smart-fix, poll node)
├── tests/                    # Test files
├── docs/                     # Documentation (Diataxis)
├── modal_app.py              # Modal deployment config
└── pyproject.toml            # Project config (pixi)
```

### File organization principles

- **Frontend**: Vanilla JavaScript (ES modules), no frameworks
- **Backend**: Python with FastAPI
- **Styling**: Modular CSS files imported by `style.css`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ericmjl/canvas-chat](https://github.com/ericmjl/canvas-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

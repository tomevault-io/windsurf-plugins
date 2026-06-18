---
trigger: always_on
description: This file provides structured guidance for AI agents (Claude, Codex, Cursor, Copilot, etc.) working on the **notewise** codebase. Read it before making changes.
---

# AGENTS.md — Guide for AI Coding Agents

This file provides structured guidance for AI agents (Claude, Codex, Cursor, Copilot, etc.) working on the **notewise** codebase. Read it before making changes.

---

## Project Summary

`notewise` is a Python CLI application that converts YouTube videos and playlists into Markdown study notes using LLM APIs. It is packaged as a `src`-layout Python project, managed with `uv`, and published to PyPI.

The repository also includes the public website under `website/` and docs source under `docs/`.

- **Entry point:** `notewise/__main__.py` → `main()` → Typer app in `cli/app.py`
- **Core pipeline:** `pipeline/core.py` → `CorePipeline`
- **Version:** `src/notewise/__init__.py` (`__version__`)
- **Python:** 3.11+ (`match` is okay)

---

## Repository Layout (Critical Paths)

```text
src/notewise/
├── _constants.py       ← ALL numeric/string defaults live here
├── config.py           ← AppSettings (Pydantic-settings), config file parsing
├── errors.py           ← ALL custom exceptions (add new ones here only)
├── logging.py          ← structlog setup; never configure logging elsewhere
├── utils.py            ← sanitize_filename and small utilities
├── cli/app.py          ← Typer commands; lazy imports inside command bodies
├── domain/             ← Pure value objects; no I/O allowed
├── llm/provider.py     ← LLMProvider wrapping LiteLLM; UsageTotals dataclass
├── llm/prompts/        ← Prompt templates as plain Python string constants
├── pipeline/core.py    ← CorePipeline; delegates heavy work to _execution.py
├── pipeline/generation.py ← StudyMaterialGenerator (chunking, LLM calls)
├── storage/repository.py  ← DatabaseRepository singleton (SQLAlchemy + SQLite)
├── ui/dashboard.py     ← Rich Live dashboard; reads PipelineEvent stream
├── ui/oauth_flow.py    ← LiteLLM OAuth/device-flow login helpers
├── ui/setup_wizard.py  ← Interactive provider/config setup wizard
├── ui/litellm_models_snapshot.json ← Bundled text-only LiteLLM setup catalog
└── youtube/            ← Transcript & metadata extraction; no LLM calls here
```

Other important root-level paths:

```text
scripts/extract_litellm_model_snapshot.py ← Refresh bundled setup model catalog
scripts/make_help.py                      ← Render grouped `make help` output from Makefile comments
docs/config/providers.mdx                 ← Provider/user-facing model docs
docs/config/configuration.mdx             ← Config key reference
docs/operate/commands.mdx                 ← CLI command reference
website/src/routes/__root.tsx             ← Website shell, metadata, JSON-LD
website/src/routes/index.tsx              ← Website landing page route
website/src/routes/install.tsx            ← Smart /install route; serves shell/PowerShell bootstraps or InstallPage
website/src/routes/sitemap[.]xml.tsx      ← Website sitemap route
website/src/components/SitemapPage.tsx    ← Browser-friendly sitemap UI and URL inventory
website/src/components/sitemapData.ts     ← Shared sitemap URL inventory for XML and browser UI
website/src/components/                   ← Website page/section components
website/src/ui/                           ← Website reusable UI primitives including Terminal and FineIcon
website/vite.config.ts                    ← TanStack Start, Nitro, Vite plugins
docs/umami.js                             ← Mintlify custom analytics script
```

---

## Inviolable Rules

These rules are enforced by CI and must never be broken:

1. **Never define custom exceptions outside `errors.py`.** Import from there in all other modules.
2. **Never add constants (numbers, strings) directly into module code.** Add them to `_constants.py` and import.
3. **Never import heavy dependencies at module top-level in `cli/app.py`.** Use the lazy-import pattern already established there.
4. **Never configure logging outside `logging.py`.** Call `configure_logging()` from the CLI entry point only.
5. **Never put I/O (network, disk) in `domain/`.** The domain layer must remain pure.
6. **Never hardcode API keys or secrets** — not even in tests.
7. **All new CLI commands** must follow the lazy-import pattern: load heavy dependencies inside the command body using the `_load_*_dependencies()` helper pattern.
8. **Never log raw LLM prompts, provider payloads, OAuth tokens, or credentials.** Redact through `logging.py`; provider failures should use summarized/redacted errors.
9. **Provider/model docs must stay snapshot-valid.** If examples or setup model availability change, update the bundled LiteLLM snapshot, README, `.env.example`, docs, and tests together.
10. **Website work uses Bun.** `website/bun.lock` is the canonical website lockfile; do not introduce npm/pnpm/yarn drift without updating docs and lockfiles together.
11. **Do not run Prettier on Mintlify docs.** Docs MDX in `docs/` is intentionally excluded because Prettier can rewrite fenced blocks inside Mintlify components into parser-breaking forms.

---

## Adding a New CLI Command

1. Define the function in `cli/app.py` using the `@app.command()` decorator.
2. Import only `typer` and `_get_console()` at call time; load pipeline/storage modules lazily inside the function body.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whoisjayd/notewise](https://github.com/whoisjayd/notewise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

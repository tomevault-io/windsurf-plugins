---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CLI Usage

- **Use `gh` CLI for all GitHub operations** -- issues, PRs, labels, runs, checks. Never use raw git commands when `gh` can do the same thing (e.g., `gh pr status` instead of manual branch tracking).
- **Use dedicated tools over Bash** -- Read instead of `cat`, Grep instead of `grep`/`rg`, Glob instead of `find`/`ls`. Reserve Bash for commands that have no dedicated tool equivalent (git, uv, gh).
- **Minimize redundant git commands** -- don't run exploratory git commands to "check state" when you already know the state from prior output. One `git status` is enough.
- **Always check docs with Context7** -- before implementing any feature or using any library API, use the Context7 MCP to look up the current documentation. Do not rely on memory or training data for library APIs -- always verify the correct, up-to-date usage.

## Project Context

byte-size-travel is a Python travel newsletter system that collects articles from RSS feeds and email sources, enriches them with AI-generated metadata via OpenAI, selects content for newsletters, generates newsletter HTML, and distributes via Amazon SES and a static website.

### Tech Stack

- **Python:** 3.12, managed with uv
- **Database:** SQLite (local article storage -- fetch and processed layers)
- **AI:** OpenAI (gpt-4.1-mini) for content enrichment and newsletter writing
- **Email:** Amazon SES for newsletter delivery
- **Data validation:** Pydantic
- **Parsing:** feedparser + BeautifulSoup for RSS/HTML, imaplib for Gmail IMAP
- **Templates:** Handlebars (Node.js) for newsletter HTML generation
- **Website:** Static HTML/CSS/JS deployed to S3

## Commands

All commands run from the project root.

```bash
# install dependencies
uv sync

# lint
uv run ruff check .

# auto-fix lint issues
uv run ruff check --fix .

# format code
uv run ruff format .

# run tests
uv run pytest tests/

# run the newsletter pipeline
PYTHONPATH=src uv run python src/main.py
```

## Principles

- **Simplicity and best practices** -- these are our guiding values. Every decision should favor the simplest correct solution that follows established conventions.
- **Always follow conventional best practices** -- for Python and every library we use. Research the current recommended approach before proposing anything. Never suggest a non-standard pattern without explicitly calling it out and justifying why. Before implementing any feature or using any library API, use Context7 to look up the current documentation and verify the correct, up-to-date usage.
- **No shortcuts or "good enough" defaults** -- if there's an established convention, use it. Don't invent project structure, naming, or patterns when a well-known standard exists. If you know the right way to do something, do it that way the first time.
- **Fix problems properly, don't suppress them** -- when a linter, type checker, or test flags an issue, fix the underlying code. Don't silence warnings with ignores, suppressions, or config overrides unless the rule is genuinely inapplicable. When using `noqa`, always document the reason in the comment (e.g., `# noqa: ARG001 -- required by framework`). A bare `noqa` without justification is not acceptable.
- **Never use falsy checks for numeric values** -- `value or None` and `if not value` silently convert `0`, `0.0`, and `""` to `None`/`False`. Use explicit checks: `value is not None`.
- **Never ignore errors** -- if a command fails, stop and investigate. Don't skip past errors, don't assume they're harmless, and don't apply hacky workarounds to make them go away. Diagnose the root cause first, then fix it properly.
- **Own all test failures** -- when a test fails during your work, fix it. Don't dismiss failures as "pre-existing" or "not caused by my changes." If the fix is genuinely complex and unrelated, file it as a GitHub issue, but try to fix it first.
- **Fix deprecation warnings immediately** -- when tests or commands emit deprecation warnings from our code, fix them right away. Warnings from third-party libraries are out of our control, but anything in our code should use the current non-deprecated API.
- **Greenfield discipline** -- don't leave unused columns, dead code, placeholder features, or "just in case" abstractions. If something isn't needed now, remove it. Every line of code should serve a current purpose.
- **Open issues for necessary but out-of-scope work** -- if implementing a feature reveals work that's needed elsewhere, open a GitHub issue with appropriate labels instead of leaving it undocumented. Necessary follow-up work must never silently fall through the cracks.
- **No ephemeral docs in version control** -- planning documents, implementation plans, and design notes are temporary artifacts. Don't commit them. The git history and PR descriptions capture the "why" permanently.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JohnathanOneal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

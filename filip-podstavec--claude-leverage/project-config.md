---
trigger: always_on
description: The always-on essentials live in [`AGENTS.md`](../AGENTS.md) ("Code
---

# Code conventions (full)

The always-on essentials live in [`AGENTS.md`](../AGENTS.md) ("Code
conventions"). This file holds the depth an agent only needs when actually
writing code in a given area — kept here, surfaced on demand, so the root
instruction file stays lean (see
[ADR 0009](adr/0009-agents-md-lean-budget-and-size-tiers.md)).

These conventions apply to code you ship in this repo AND are what the stack
documents for other repos via
[`templates/AGENTS.md.example`](../templates/AGENTS.md.example).

## Naming

The inline rule is `Name to fit in` (`AGENTS.md` → "Write less, fit in"). The
discipline behind it, and why it is detect-and-conform rather than a house style,
is in [ADR 0010](adr/0010-naming-detect-and-conform-over-house-style.md). The
mechanics:

**Casing / separator — detect, then conform.** There is no stack-wide "correct"
style; the correct style is whatever the surrounding code already uses. A model's
language default (Python → `snake_case`, JS → `camelCase`) is a *prior*, not a
license to impose. Before naming anything new:

- **Scan first.** Look at sibling files and nearby identifiers of the *same kind*
  to read off the convention. `grep` a few existing definitions if unsure.
- **Match per kind.** Casing legitimately differs by kind even within one repo —
  `PascalCase` types, `camelCase`/`snake_case` functions and locals,
  `UPPER_SNAKE` constants, `kebab-case` files/CLI flags. Conform each kind to its
  own neighbours, not to a single global rule.
- **Local over global.** In a repo with inconsistent history, match the *local
  module* you're editing over the repo-wide majority — fitting the immediate
  context beats a "correct" name that clashes with everything around it.
- **Idioms only if the repo uses them.** Predicate prefixes (`is_`/`has_`/
  `should_`), hungarian-ish suffixes, `_async` markers, etc. — adopt them only
  when the surrounding code already does. Don't import a convention the repo
  never chose.

**Granularity / clarity — universal.** Independent of the repo, a name states
intent at the right altitude:

- **Too vague** (`get()`, `data()`, `handle()`, `process()`, `tmp`) forces the
  reader to open the body to learn what it does.
- **Too verbose / leaking** (`getting_data_from_mobile()`,
  `user_list_array_final2`) bakes implementation detail or history into the name,
  so it reads as noise and goes stale when the internals change.
- **Right** names the *what/why* at the call site's level of abstraction:
  `fetch_mobile_profile()`, `pending_invoices`, `is_expired`. If a good name is
  hard to find, the unit is often doing too much — that's a design signal, not a
  naming problem.

## Repo layout

```
agents/                       Claude Code subagents (Markdown + YAML frontmatter)
.codex/agents/                Codex subagents (TOML; generated from agents/)
skills/                       Cross-tool skills (SKILL.md, agentskills.io spec)
commands/                     Claude Code slash commands
hooks/hooks.json              Claude Code hook config — paths point at scripts/hooks/
.codex/hooks.json             Codex hook config (template; install-codex resolves paths)
.codex/config.toml            Codex sandbox/approval policy
scripts/hooks/                Hook shell scripts, shared by both tools
scripts/                      Installers, generators, version checks, smoke-plugin.sh
statusline/                   Portable statusline script
assets/                       README banner (banner.svg) + static image assets
claude-md-snippets/           Opt-in CLAUDE.md / AGENTS.md routing rules (installable via /init-repo)
templates/                    Per-repo AGENTS.md examples + structured-logging starter kits
agents-docs/, commands-docs/  Per-dir docs that can't live inside agents/ or
                              commands/ because Claude Code's plugin loader
                              registers every *.md as a phantom — see
                              tests/test_agent_command_frontmatter.py
docs/adr/                     Architecture Decision Records (numbered, immutable; /adr-new bootstraps)
docs/sessions/                Distilled session logs (/session-log writes one at end of session)
docs/specs/                   Design specs (current and historical)
workflows/                    End-to-end prose guides combining skills/hooks/conventions
bench/archive-token-savings-thesis/
                              Frozen evidence of the v0.x token-savings experiment
                              that motivated the v1.0 pivot. Don't delete.
```

## AIDEV-* anchor deadlines (optional)

`AIDEV-TODO` and `AIDEV-QUESTION` accept an optional ISO-8601 deadline:

```python
# AIDEV-TODO(by: 2026-08-01): replace the polling loop with webhooks
# AIDEV-QUESTION(by: 2026-07-15): is the encoding always UTF-8 here?
```

`/stack-check`'s anchor walk parses the date and reports overdue items
separately from age-based "stale" items, so deadlines have actual teeth.
Without a deadline, the same anchor falls under the age-based check
(fresh / aging / stale at 30 / 90 days).

## Structured JSON-lines logging

For application code that emits logs an agent will later need to read:

```json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Filip-Podstavec/claude-leverage](https://github.com/Filip-Podstavec/claude-leverage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

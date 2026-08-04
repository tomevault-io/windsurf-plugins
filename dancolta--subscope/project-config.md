---
trigger: always_on
description: Validates the plugin manifest:
---

# subscope — Contributor Guide

This file is for **contributors working on the codebase**, not plugin users. Plugin users want [README.md](README.md). Plugin behavior is defined in [`skills/`](skills/), not here. (`claude plugin validate` flags this file as "not loaded as plugin context" — that's correct, by design.)

If you opened a clone and asked Claude to help you contribute, this is the orientation document.

---

## What this codebase is

A Python engine + Claude Code skills:

- **`engine/subscope/`** — Python. Fetches Reddit via public JSON, runs regex + optional LLM gates, scores survivors, writes to SQLite, prints JSON to stdout. Stdlib + pyyaml + optional `openai`, `notion-client`.
- **`skills/*/SKILL.md`** — 15 user-invocable Claude Code skills. Each one is a single Markdown file that tells Claude how to orchestrate a workflow (Notion sync via MCP, Obsidian write via MCP, Playwright blog refresh, etc.). The Python engine does no MCP work — the skill layer does.
- **`config/`** — YAML defaults: weights, default subreddits, default keywords, scoring caps. Public users override by writing to `~/.config/subscope/`.
- **`presets/`** — 4 starter bundles (b2b-saas-founder, agency-owner, indie-hacker, consultant) for users who don't want to run `/subscope-onboard`.
- **`assets/`** — README hero GIF + the Python+Pillow render script.

The engine is intentionally separable: you could pipe its JSON output to any orchestrator, not just Claude Code.

---

## File layout

```
.
├── .claude-plugin/plugin.json    # plugin manifest (required by Claude Code)
├── engine/
│   ├── subscope/
│   │   ├── cli.py                # all CLI subcommands (fetch-score, status, op-vet, ...)
│   │   ├── lib/                  # the engine modules
│   │   │   ├── store.py          # SQLite + XDG paths + enrichment cache helpers
│   │   │   ├── score.py          # gate + score + selection
│   │   │   ├── reddit.py         # public-JSON fetcher
│   │   │   ├── classify.py       # OpenAI-compat bulk LLM grader
│   │   │   ├── author_vet.py     # OP karma/age/audience pre-gate
│   │   │   ├── discover.py       # live subreddit discovery for /onboard T5 (recall stage)
│   │   │   ├── archetype_map.py  # 6 archetypes, fallback seed for /onboard + /profile
│   │   │   ├── profile_synth.py  # 8-Q + 3-Q config synthesis
│   │   │   ├── obsidian_sync.py  # weekly pulse digest builder
│   │   │   ├── enrich.py         # DataForSEO + Firecrawl conditional consumers
│   │   │   ├── net.py            # SSRF guard + certifi-aware SSL context
│   │   │   ├── slack.py          # optional webhook push
│   │   │   ├── tune_engine.py    # /tune ranker back-prop
│   │   │   └── output.py         # markdown + table renderers
│   │   └── prompts/              # system prompts (classify, profile_synth)
│   ├── scripts/                  # one-shot helpers (write_dataforseo_config, write_firecrawl_config, notion_admin, ...)
│   └── tests/                    # pytest
├── skills/                       # 15 SKILL.md files, one per pattern
├── config/                       # default YAML (subreddits, keywords, weights, presets)
├── presets/                      # 4 starter bundles
├── assets/                       # hero.gif + render_hero.py
└── docs/                         # setup-notion.md (public only)
```

---

## Conventions

These are non-negotiable for any PR:

- **No em dashes** in any user-facing text (chat output, Notion writes, error messages). Use commas or restructure. The engine output is em-dash-free; preserve that. `engine/tests/test_no_em_dashes.py` enforces it.
- **Parameterized SQL only.** Every `conn.execute()` must use `?` placeholders. f-string SQL is a defect.
- **`chmod 600` on every config + DB file.** Atomic creation via `os.open(path, O_WRONLY|O_CREAT|O_TRUNC, 0o600)` — never `open()` then `chmod()` (umask race).
- **XDG-compliant paths.** Config at `~/.config/subscope/` (or `$XDG_CONFIG_HOME/subscope/`), data at `~/.local/share/subscope/`. Override via `SUBSCOPE_CONFIG` / `SUBSCOPE_DATA` env for tests.
- **Reddit username validation.** Any value interpolated into a `reddit.com/user/<x>/` URL must pass `reddit._safe_username()` regex first (defuses path-segment injection).
- **No new shell=True subprocess calls.** Use `subprocess.run([..., args], shell=False)` with list args. The engine has zero `shell=True` calls today; keep it that way.
- **SSRF guard.** Any user-configurable URL (LLM endpoint, Slack webhook, future adapters) must validate scheme + host before the HTTP call. See `classify._validate_base_url()` for the pattern.
- **No telemetry, ever.** No analytics, no error reporting, no usage pings. If you need to send anything off the user's machine, it must be opt-in with a one-time stderr banner.

---

## Adding a new pattern skill

The 8 patterns share one engine (`fetch-score --mode <pattern>`). Adding a pattern is ~30 minutes:

1. Add pattern keywords: `config/keywords-<pattern>.yml`
2. Add cap in `config/weights.yml` under `pattern_caps`
3. Add the mode to `VALID_MODES` in `engine/subscope/cli.py`
4. Add an emoji in `PATTERN_EMOJI` in `cli.py`
5. Create `skills/<pattern>/SKILL.md`:

```markdown
---
name: <pattern>
description: One-paragraph description. Triggers on "<pattern>", "/subscope-<pattern>", "...".
allowed-tools: Bash, Read, Write

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dancolta/subscope](https://github.com/dancolta/subscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->

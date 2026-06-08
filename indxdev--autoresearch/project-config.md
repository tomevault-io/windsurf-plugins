---
trigger: always_on
description: This repository is an **Auto Research** template. Inside GitHub Actions, you
---

# Project guide for Claude (Auto Research)

This repository is an **Auto Research** template. Inside GitHub Actions, you
(Claude Code) run the **research half** of each section: you web-search the
topic and return a **schema-validated JSON object** as structured output. A
small Python script then deterministically turns that JSON into GitHub Issues —
**one Issue per item** (each news item, paper, or hypothesis is its own Issue).

So the split is:

- **You (Claude Code Action):** the open-ended, trial-and-error part — search,
  read, reason, and emit structured JSON. **You do not create Issues or files.**
  Claude is the **main** engine. If a run has no Claude credential but an OpenAI
  one (`OPENAI_API_KEY` / `CODEX_ACCESS_TOKEN`), the local composite action
  `.github/actions/agent` falls back to **OpenAI Codex** (`openai/codex-action`)
  as the **sub**, normalising its reply with `scripts/extract_json.py` into the
  same schema-shaped JSON. Claude always wins when both are present.
- **Python (`scripts/publish_section.py`):** the deterministic part — iterates
  the array, formats each item into Markdown, opens one Issue per item, writes
  optional files, and emits one Slack line (and, when configured, one email) per
  item. Each line's **primary** link is a public URL — the item's page on the
  published Pages site (resolved live via `scripts/site_url.py`), else the
  GitHub Issue. The Issue is shown as a secondary `↳` link when the site page
  leads, and becomes the primary only as a fallback. The curated source URL (the
  paper/article) stays inside the Issue body, not the notification.
- **One combined send per run.** The four section jobs run in parallel and would
  each fire their own Slack post + email. By default (`COMBINE_NOTIFICATIONS` ≠
  `false`) every publisher routes its lines through `scripts/notify.py`, which —
  seeing `NOTIFY_SPOOL_DIR` set — **spools** each block to `.notify/` instead of
  sending. Each section uploads that as a `notify-<section>` artifact; a final
  `notify` job downloads them all and runs `scripts/notify_combined.py` to fold
  every section into **one Slack post and one email** for the whole run (ordered
  Latest → Takes → Foundations → Site Watch). Pin `COMBINE_NOTIFICATIONS=false`
  to go back to per-section sends.

## What matters

- **Use real web search.** Ground every item in sources you actually opened via
  the WebSearch / WebFetch tools. Prefer arXiv, Semantic Scholar, top venues,
  and reputable blogs. **Never fabricate** papers, authors, dates, or URLs. If
  unsure, leave a field blank or describe the type of work — do not invent a
  citation.
- **Return ONLY the JSON required by the `--json-schema`.** Do not wrap it in
  prose or Markdown. The schema is the contract; the Python publisher renders it.
- **Respect the configuration** in the prompt: output language (write the text
  fields in that language), research topic, and the per-section instructions.
- Read `config/research_topics.md` for the lab's topics/notes, and
  `config/prompts/` for per-section guidance.
- **Security:** never print or echo secrets, tokens, or the Slack webhook URL.

## Domains: one lens per run (research / tech / business / hobby / finance)

A run is not limited to academic research. The **`select` job** runs first and
picks ONE **domain** — `research`, `tech`, `business`, `hobby`, or `finance` —
each described by a plain Markdown guide under `config/domains/<domain>.<lang>.md`
(English + 日本語). The three section jobs then `needs: select` and share that
choice. Whatever the domain, every run produces the SAME three generic layers,
and the chosen domain guide says what each layer means in that domain:

- **Foundations** (the `related` section) — the unwavering, slow-moving facts.
- **Latest** (the `news` section) — recent, concrete developments.
- **Takes** (the `hypotheses` section) — interpretations / theses, each testable.

How the domain + language are decided (you usually set NO Variables):

- **Domain:** default `auto` — a tiny Claude picker reads `config/domains/index.md`
  and the topic and chooses. Pin it with the `RESEARCH_DOMAIN` Variable if you want.
- **Language:** `OUTPUT_LANGUAGE` when set; otherwise the picker infers it from the
  topic.
- **Sticky:** `scripts/select_domain.py` hashes the effective topic and REMEMBERS
  `{domain, lang}` for that hash on the `auto-research-state` orphan branch, so the
  choice only changes when the topic changes. The picker step runs only when a fresh
  choice is actually needed (or never, with no Claude key → sensible defaults).

When you are the picker, return ONLY `{"domain": ..., "lang": ...}`. When you are a
section, `Read` the named `config/domains/<domain>.<lang>.md` FIRST and follow it.

## How a run is wired (per section, in parallel)

1. `select` job: `scripts/select_domain.py prepare` decides if a fresh pick is
   needed; if so a small Claude step picks the domain + language; `finalize`
   remembers it on `auto-research-state`. Exposes `domain` / `lang` outputs.
2. `news` / `hypotheses` / `related` are independent jobs (`needs: select`), each
   gated by its flag, all sharing the run's domain + language.
3. Context step: `scripts/existing_context.py` summarises Issues from prior runs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [INDXDev/autoresearch](https://github.com/INDXDev/autoresearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->

---
trigger: always_on
description: Use when building or maintaining a personal LLM-powered knowledge base. Triggers: ingesting sources into a wiki, querying wiki knowledge, 'add to wiki', 'what do I know about', or any mention of 'LLM wiki' or 'Karpathy wiki'.
---


# Karpathy LLM Wiki

Build and maintain a personal knowledge base using LLMs. You manage two directories: `raw/` (immutable source material) and `wiki/` (compiled knowledge articles). Sources go into raw/, you compile them into wiki articles, and the wiki compounds over time.

Core ideas from Karpathy:

- "The LLM writes and maintains the wiki; the human reads and asks questions."
- "The wiki is a persistent, compounding artifact."

## Architecture

Three layers, all under the user's project root:

**raw/** — Immutable source material. You read, never modify. Organized by topic subdirectories (e.g., `raw/machine-learning/`).

**wiki/** — Compiled knowledge articles. You have full ownership. Organized by topic subdirectories, one level only: `wiki/<topic>/<article>.md`. Contains two special files:

- `wiki/index.md` — Global index. One row per article, grouped by topic, with link + summary + Updated date.
- `wiki/log.md` — Append-only operation log.

**SKILL.md** (this file) — Schema layer. Defines structure and workflow rules.

Templates live in `references/` relative to this file. Read them when you need the exact format for raw files, articles, archive pages, or the index.

### Initialization

Triggers only on the first Ingest. Check whether `raw/` and `wiki/` exist. Create only what is missing; never overwrite existing files:

- `raw/` directory (with `.gitkeep`)
- `wiki/` directory (with `.gitkeep`)
- `wiki/index.md` — heading `# Knowledge Base Index`, empty body
- `wiki/log.md` — heading `# Wiki Log`, empty body

If Query cannot find the wiki structure, tell the user: "Run an ingest first to initialize the wiki." Do not auto-create.

---

## First-run bootstrap

Before the first invocation of any helper under `tools/` in a session (`tools/search.py`, `tools/graph.py`, `tools/dashboard.py`, `tools/watcher.py`), verify the Python deps are installed.

### Check

Run this once per session and remember the result:

```bash
python -c "import rank_bm25, networkx, textual, watchdog" 2>&1
```

- **Exit 0** — deps are installed. Proceed.
- **`ModuleNotFoundError`** — deps are missing. Go to Install.

### Install

1. Locate the skill root. If the working directory is a vault (contains `raw/` + `wiki/`), the skill lives elsewhere — typical locations:
   - `~/.claude/skills/second-brain/`
   - `~/.config/agent-skills/second-brain/`
   - wherever `npx skills add dawnn07/second-brain` dropped it

2. Tell the user exactly once:

   > This skill needs one-time Python setup (`pip install -e .` in the skill directory). This installs `rank-bm25`, `networkx`, `textual`, `watchdog`, and optional semantic-search libraries. Proceed?

3. On confirmation, run from the skill root:

   ```bash
   pip install -e .
   ```

   If `pip` is not available, fall back to:

   ```bash
   python -m pip install -e .
   ```

4. Re-run the check. If it still fails, report the error verbatim and stop — do not attempt further fallbacks (virtualenv creation, conda, etc.); that's the user's call.

### When to skip

- The user explicitly says "don't install anything" — fall back to direct `wiki/index.md` reads for Query, and tell the user graph/dashboard/watcher are unavailable until they install.
- The wiki has fewer than ~30 articles — Query can read `index.md` directly without search. Offer to skip install until the vault grows.

### What not to do

- Do not re-prompt for install within the same session. Cache the decision.
- Do not auto-install without asking. Always get consent first.
- Do not fall back to `pip install --user` or `sudo pip install`.

---

## Ingest

Fetch a source into raw/, then compile it into wiki/. Always both steps, no exceptions.

### Fetch (raw/)

1. Get the source content. For local files, use the Read tool. For URLs, use WebFetch. If neither can reach the source, ask the user to paste it directly.

2. Pick a topic directory. Check existing `raw/` subdirectories first; reuse one if the topic is close enough. Create a new subdirectory only for genuinely distinct topics.

3. Save as `raw/<topic>/YYYY-MM-DD-descriptive-slug.md`.
   - Read `references/raw-template.md` for the exact format.
   - Slug from source title, kebab-case, max 60 characters.
   - Published date unknown → omit the date prefix from the file name (e.g., `descriptive-slug.md`). The metadata Published field still appears; set it to `Unknown`.
   - If a file with the same name already exists, append a numeric suffix (e.g., `descriptive-slug-2.md`).
   - Include metadata header: source URL or local file reference, collected date, published date.
   - Preserve original text. Clean formatting noise. Do not rewrite opinions.

### Compile (wiki/)

Read `references/article-template.md` for the exact article format.

Determine where the new content belongs:

- **Same core thesis as existing article** → Merge into that article. Add the new source to Sources and Raw fields. Update affected sections.
- **New concept** → Create a new article in the most relevant topic directory. Name the file after the concept, not the raw file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dawnn07/second-brain](https://github.com/dawnn07/second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

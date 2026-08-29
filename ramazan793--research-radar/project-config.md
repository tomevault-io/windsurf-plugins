---
trigger: always_on
description: Daily arXiv digest daemon. It fetches each day's new papers in your chosen categories,
---

# Research Radar

Daily arXiv digest daemon. It fetches each day's new papers in your chosen categories,
scores them against *your* research interests, deep-reads the most promising ones, and
writes an HTML/Markdown digest (with an optional Telegram ping). Field-agnostic: what to
care about lives entirely in `context/research_context.md`, not in the code. Typically
run from cron on an always-on machine.

## Pipeline

```
cron (daily; run time set by your crontab, digest dates use timezone_offset_hours)
  -> src/main.py
       1. Load config.yaml + context/research_context.md
       2. Fetch papers: per category, RSS (today) + API lookback window, merged
          - API lookback always runs (backfills missed runs/weekends; dedup makes it cheap)
          - Dedup across categories by arxiv_id
          - Reports per-category fetch outages (RSS+API both errored) distinctly from quiet days
       3. Filter against data/seen_papers.json (skip already-seen)
       4. PASS 1 — Abstract scoring (scorer.py)
          - Batches of batch_size papers -> SCORING backend/model (cheap/fast)
          - Title + abstract only, scored 1-10 against research_context.md
          - Returns JSON: [{arxiv_id, score, reason}]. Failures isolate per-PAPER: each
            retry re-asks only the still-unscored papers; any left over after retries are
            recorded (with a reason) to data/digests/YYYY-MM-DD_failures.json, not dropped
       5. PASS 2 — Deep read (deep_reader.py)
          - Papers scoring >= deep_read_min_score (default 7), capped at max_deep_reads
          - Downloads full PDF + extracts text via pymupdf (deterministic, no model)
          - Sends full paper to DEEP_READ backend/model (strong/slow)
          - Returns: summary, what_it_opens, key_insights, ideas, limitations,
            refined_score, verdict
          - Saved to data/digests/YYYY-MM-DD_deep.json
          - Retryable failures (PDF download/extract error, or model failure after retries)
            are held for retry like scoring failures; text-too-short is skipped permanently
       6. Generate HTML + markdown digests via Jinja2 templates
       7. Update seen_papers.json (atomic write; after the digest; skipped on --dry-run).
          Papers that failed scoring OR deep-read are held back (left unseen) so the next
          run retries them
       8. Optional: Telegram notification with top papers (clickable arXiv links)
          + digest link  (skipped if Telegram is not configured)
```

Both model passes go through the model-agnostic `backends/` layer. `main.resolve_task()`
reads the `tasks:` section of `config.yaml` and resolves a `(backend, model, effort)`
triple **per task** — so scoring can run on a cheap model and deep-read on a strong one,
even across different providers.

## File Map

```
src/
  main.py              # Entry point + orchestrator. resolve_task() picks per-task
                       #   backend/model/effort; accumulates token usage. Supports --dry-run.
  arxiv_fetcher.py     # RSS + API fetching, dedup, date filtering
  scorer.py            # Pass 1: batched abstract scoring; salvages partial batches, isolates
                       #   per-paper failures, retries only the unresolved papers
  deep_reader.py       # Pass 2: PDF download + pymupdf text extraction, then deep analysis
  digest_generator.py  # Jinja2 rendering of HTML/MD digests + index.html
  telegram_bot.py      # Optional Telegram notifications (digest, no-papers, errors)
  backends/            # Model-agnostic inference layer
    base.py            # LLMBackend ABC + complete(); Usage / CompletionResult dataclasses;
                       #   shared JSON extract/validate helpers (backend-agnostic)
    _cli.py            # Robust CLI executable resolution (cron-safe PATH lookup)
    codex.py           # Codex CLI backend (codex exec --json; ChatGPT plan credits)
    claude.py          # Claude Code CLI backend (claude -p --output-format json; Claude.ai plan)
    openai_compatible.py # Any OpenAI /chat/completions endpoint (Ollama/vLLM/OpenRouter/
                       #   OpenAI/...). Also the heavily-commented "write your own" template.
    __init__.py        # get_backend(name, config) factory + backend registry
  codex_cli.py         # Deprecated shim (re-exports CLI resolution; use backends/ instead)

config.yaml            # backends + per-task models + thresholds + categories + timezone
context/
  research_context.example.md  # generic template — copy it and edit
  research_context.md          # YOUR research profile + scoring guide. User-provided
                               #   (copied from .example, gitignored). Drives all scoring.
templates/
  digest.html.j2       # HTML digest (score badges, deep-analysis boxes)
  digest.md.j2         # Markdown digest
  index.html.j2        # Archive index page
data/
  seen_papers.json     # Dedup state: {arxiv_id: {title, date_first_seen, score}}
  digests/             # Generated digests: YYYY-MM-DD.html, .md, _deep.json,
                       #   _failures.json (papers that couldn't be scored), + index.html
  cron.log             # Cron output log

pyproject.toml         # uv app (package=false); deps: requests, feedparser, python-dotenv,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ramazan793/research-radar](https://github.com/ramazan793/research-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

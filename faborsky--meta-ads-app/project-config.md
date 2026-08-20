---
trigger: always_on
description: Python CLI for Facebook & Instagram ads via Marketing API **v25.0**. Version 2.4.0, 47 commands. Czech user docs in [README.md](README.md).
---

# Meta Ads App — CLI for the Meta Marketing API

Python CLI for Facebook & Instagram ads via Marketing API **v25.0**. Version 2.4.0, 47 commands. Czech user docs in [README.md](README.md).

## Setup

```bash
<APP_DIR>/run.sh <command> [flags]        # activates venv automatically
# or: source <APP_DIR>/venv/bin/activate && python <APP_DIR>/meta_ads_cli.py <command>
```

Credentials in `.env`: `META_ACCESS_TOKEN` (60-day long-lived), `META_AD_ACCOUNT_ID` (`act_…`), `META_PAGE_ID`, `META_APP_ID`, `META_APP_SECRET`. Override account per-command: `--account-id act_XXX` (global flag, before the subcommand).

## Code structure

- `meta_ads_cli.py` — thin entrypoint + backward-compat re-exports (`_api_call`, `META_AD_ACCOUNT_ID`, …) for scripts that `import meta_ads_cli as cli`
- `metaads/api.py` — engine: env/config, `_api_call` (Bearer auth, redacted errors, retries — transient retry GET-only), rate-limit header parsing + persistent usage guard (`.usage/`, hard-stop ≥95 %, per-account), token-expiry warning (daily cached, never fatal), `mutate()` with validate_only dry-run, budget currency guard (zero-decimal currencies refused)
- `metaads/formatting.py` — output helpers (no API logic)
- `metaads/lint.py` — preflight checks: text lengths/style (Meta editorial policy), URL, CTA, image dimensions
- `metaads/cli.py` — argparse wiring; `_cmd()` helper = parser/dispatch parity by construction
- `metaads/commands/*.py` — one module per domain (account, auth, campaigns, adsets, ads, creatives, media, insights, pulse, activity, conversions, targeting); no shared mutable module state
- `scripts/check_docs_consistency.py` — CLI ↔ docs ↔ skill consistency gate
- `tests/` — offline pytest suite (no credentials needed): `venv/bin/python -m pytest tests/`; dev deps in `requirements-dev.txt`

## Commands (47, grouped)

- **Account & token**: `account`, `pages`, `api-limits`, `token-info`, `token-extend`
- **Campaigns**: `campaigns`, `campaign-detail`, `campaign-create`, `campaign-update`, `campaign-duplicate`, `campaign-delete`, `budget-schedule`
- **Ad sets**: `adsets`, `adset-detail`, `adset-create`, `adset-update`, `adset-duplicate`, `adset-delete`
- **Ads**: `ads`, `ad-detail`, `ad-review`, `ad-create`, `ad-update`, `ad-duplicate`, `ad-delete`
- **Creatives & media**: `creatives`, `creative-detail`, `creative-create`, `creative-clone`, `creative-from-post`, `creative-from-ig`, `ig-media`, `preview`, `creative-delete`, `image-upload`, `video-upload`
- **Insights & analysis**: `insights`, `insights-report`, `pulse`, `activities`
- **Conversions (read-only)**: `pixels`, `custom-conversions`
- **Targeting search**: `interest-search`, `interest-suggest`, `interest-validate`, `geo-search`, `locale-search`

Full flags: README.md command tables, or `--help` per command.

## Safety

- **Writes default to dry-run** (`execution_options=["validate_only"]`); `--confirm` executes. Endpoints without validate_only (`/copies`, `/budget_schedules`) print a plan and make no call without `--confirm`.
- All create/duplicate commands default to `--status PAUSED` / PAUSED copies — nothing goes live automatically.
- `campaign/adset/ad-delete` refuse non-PAUSED/ARCHIVED entities without `--force`; prefer ARCHIVED (half-way trash, still queryable). `creative-delete` has no status brake (creatives have no PAUSED state) — Meta itself refuses to delete an in-use creative.
- Exception to dry-run: `image-upload`/`video-upload` write directly (media library only, no spend risk).
- Preflight lint runs locally before creative writes (text lengths, CAPS/punctuation/emoji, URL, CTA, image dimensions).
- Listings hide `effective_status: DELETED` rows by default.
- Always use `--json` when parsing output programmatically.

## ⚠️ Critical for automation (read before scripting writes)

- **DELETE is permanent; ARCHIVED is the trash can.** Deleted objects stay readable by ID (stats 28 days) and linger in edges — the CLI filters them.
- **Ad review is asynchronous** (typically <24 h). Creative/targeting changes trigger re-review; budget/bid/schedule changes don't. Check with `ad-review` after creating/swapping creatives. Paused ads stay paused after review.
- **Creatives are immutable** — "editing" = create new creative + swap on ad (`ad-update --creative-id` or `creative-clone --swap-on-ad`).
- **New creatives get Advantage+ enhancements by default** (no `degrees_of_freedom_spec` = Meta's defaults, typically ON). `--no-enhancements` on creative-create/from-post/from-ig/clone sends all 14 features as OPT_OUT. `url_tags` is a top-level creative field — `creative-clone` carries it since 2.3.0; any manual rebuild must too.
- **Videos > 100 MB upload chunked automatically** (single multipart POST 413s on large files); `--chunked` forces it. Chunk transfers are the only writes allowed to retry transient errors (offset-addressed = idempotent).
- **Rate limits are hourly BUC windows** (Limited access tier: 300 + 40×active ads calls/h; Full access: 100k + 40×active ads). Usage only via response headers; CLI persists it in `.usage/` and hard-stops ≥95 % per account. Don't parallel-fan-out API calls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [faborsky/meta-ads-app](https://github.com/faborsky/meta-ads-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->

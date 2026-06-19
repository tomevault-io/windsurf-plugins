---
trigger: always_on
description: Personal finance system. Drive vault → SQLite → HTML dashboard. Three skills do the work — `fetch-bank-data` pulls fresh transactions from Hapoalim + Cal into Drive `dump/`; `sync-finance-data` ingests everything in the vault into SQLite; `render-finance-dashboard` reads SQLite into the HTML. Everything else is data, templates, and documentation.
---

# findash

Personal finance system. Drive vault → SQLite → HTML dashboard. Three skills do the work — `fetch-bank-data` pulls fresh transactions from Hapoalim + Cal into Drive `dump/`; `sync-finance-data` ingests everything in the vault into SQLite; `render-finance-dashboard` reads SQLite into the HTML. Everything else is data, templates, and documentation.

## First principles

These shape every decision in this project. Re-read them when you're about to write code.

1. **Public-project hygiene.** This repository is public. Keep committed code, docs, examples, and prompts generalized so they work for any user. Never include a real user's account numbers, balances, transaction amounts, counterparties, credentials, tokens, Drive IDs, personal identifiers, or local secret values. Provider names are fine when documenting supported integrations; user-specific financial details are not. Never print, paste, upload, or otherwise share secrets from `.secrets/`, `rclone.conf`, local databases, downloaded statements, or generated dashboards.

2. **Judgment over scripts.** Codex's reasoning is the asset, not a parser script. Don't write categorization rules, don't pattern-match filenames mechanically, don't hard-code "if counterparty contains X then category is Y". The example: a large outflow to a brokerage you own is *not* an expense — it's a transfer. Only Codex can see that, because Codex reads both the bank statement and the brokerage's deposit confirmation in the same session. Scripts can't.

   Mechanical work (parsing XLSX bytes, executing SQL, decrypting a PDF with a known password, calling `rclone`) is fine as a script. *Interpretation* of what the data means is always done by Codex.

3. **One topic per file.** A skill describes a flow; it never repeats schema details. The schema doc never repeats the Drive layout. If you're about to write the same fact in two places, stop and decide which file owns it.

4. **Instruct, don't hardcode.** Tell Codex what tables exist and what each doc type generally looks like. Don't dictate the SQL queries to run or the regexes to match. The exception is artifacts that can only be code: the SQL schema (`init-db.sql`), the XLSX byte-parser (`scripts/xlsx_to_rows.py`), the HTML template, the CSS.

5. **Money as integers.** All amounts stored as `amount_minor INTEGER`. Multiply on the way in, divide on the way out. Never `REAL` for money.

6. **Audit trail is non-negotiable.** Every row in every fact table has a `source_doc_id` pointing back to `documents`. If a row can't cite its source, it doesn't get inserted.

7. **Idempotency.** Running any skill twice on the same Drive state must be a no-op. Dedup keys: `documents.drive_id` for files; `(account_id, as_of, component)` for balances; `(period_start, period_end, employer)` for payslips.

## What lives where

```
~/findash/
├── AGENTS.md                 ← you are here
├── .gitignore
├── rclone.conf               ← OAuth token, chmod 600, gitignored
├── .secrets/
│   ├── drive                 ← root_folder_id=… for the Drive vault, chmod 600
│   ├── pdf-passwords         ← <pattern>=<password> lines, chmod 600
│   ├── telegram              ← bot_token=… / chat_id=… for dashboard delivery
│   ├── hapoalim              ← user_code=… / password=… for fetch-bank-data
│   └── cal                   ← username=… / password=… for fetch-bank-data
├── .Codex/skills/
│   ├── sync-finance-data/SKILL.md
│   ├── render-finance-dashboard/SKILL.md
│   └── fetch-bank-data/SKILL.md
├── docs/
│   ├── sqlite-schema.md      ← schema conventions + example queries
│   ├── drive-layout.md       ← Drive folder structure (ID lives in .secrets/drive)
│   ├── doc-types.md          ← what each kind of document contains + judgment calls
│   ├── design-system.md      ← the booky aesthetic
│   └── inspiration/          ← reference images
├── templates/
│   ├── dashboard.html.tpl    ← HTML shell with {{PLACEHOLDER}}s
│   ├── styles.css            ← inlined into dashboard.html at render time
│   ├── charts.js             ← inlined into dashboard.html at render time
│   └── vendor/               ← gitignored; produced by scripts/bundle-assets.py
│       ├── chart.umd.min.js
│       ├── chartjs-adapter-date-fns.bundle.min.js
│       └── fonts-inline.css  ← EB Garamond + Cormorant Garamond as base64 woff2
├── data/
│   └── finance.db            ← local SQLite, synced to Drive
├── inbox/                    ← transient downloads from Drive
├── output/
│   └── dashboard.html        ← single self-contained file (CSS + JS + fonts inlined)
└── scripts/
    ├── init-db.sql           ← schema definition
    ├── xlsx_to_rows.py       ← stdlib-only XLSX → JSON parser
    ├── bundle-assets.py      ← one-shot vendor script for templates/vendor/
    ├── fetch_bank.js         ← Puppeteer wrapper around israeli-bank-scrapers (Hapoalim + Cal)
    ├── package.json          ← npm deps for fetch_bank.js (`cd scripts && npm install`)
    └── node_modules/         ← gitignored
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ya5huk/findash](https://github.com/ya5huk/findash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->

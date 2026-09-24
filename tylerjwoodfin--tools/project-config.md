---
trigger: always_on
description: Import bank, card, Venmo, and other CSVs into self-hosted Sure (`https://sure.tyler.cloud`).
---

# Finance parser

Import bank, card, Venmo, and other CSVs into self-hosted Sure (`https://sure.tyler.cloud`).

`main.py` currently automates **Venmo** (latest `VenmoStatement*.csv` → previous calendar month). For other institutions, import with the Sure API using the same conventions below. Do **not** update a spreadsheet. Do **not** invent a second budget system.

Household facts (pay, property, debt, last month’s numbers) live **only** in the private file `~/git/backend/finances/AGENTS.md`. Do **not** copy balances, salary, addresses, loan rates, or people’s names into this file.

## When to use this

- **Import:** the user asks to import transactions, load a statement CSV, or sync a download into Sure / the budget — Venmo, EverBank, Robinhood Credit Card, Amazon orders, or another CSV they name. If they point at a specific file, use that file. Otherwise pick the newest matching download in `~/Downloads`.
- **Monthly recap:** the user asks to summarize finances, recap a month, or review Sure spending. Read `~/git/backend/finances/AGENTS.md` first, then follow **Monthly recap** below. After the recap, update that file’s **Last snapshot**.

## Venmo (parser)

```bash
cd ~/git/tools/finance_parser
python3 main.py              # previous calendar month → Sure Venmo
python3 main.py --dry-run
python3 main.py --file /path/to/VenmoStatement_….csv
python3 main.py --month YYYY-MM
```

Looks for the newest `~/Downloads/VenmoStatement*.csv`. If none exist, tell the user to download the monthly statement from Venmo (Activity → statements) into Downloads. Do not open a file picker.

Exit `0` with a create/update/skip summary means it worked.

## Other CSVs (Sure API)

`main.py` is Venmo-only until it grows. For anything else, import via the LAN Sure API (do not wait on a parser rewrite unless the user asks). Typical Downloads globs:

| Source | Typical file | Sure account |
|--------|----------------|--------------|
| Venmo | `VenmoStatement*.csv` | `Venmo` |
| EverBank | `Transactions_*.csv` (cols: Date, Check#, Transaction Type, Description, Debits(-), Credits(+)) | `Everbank` |
| Robinhood CC | UUID-named CSV with `Cardholder,Amount,Points,Balance,Status,Type,Merchant,Description` | `Robinhood Credit Card` |
| Amazon orders | `amazon-orders-*.csv` (line items) | `Amazon Credit Card [Ally Checking]` |

List accounts with `GET /api/v1/accounts` and match by name (do not hard-code IDs).

### CSV shape notes

- **EverBank:** `Debits(-)` are usually already negative (money out → Sure `expense`); `Credits(+)` are money in → `income`. Parse `$` and commas. Date is `MM/DD/YYYY`.
- **Robinhood CC:** Keep `Status == Posted`. Skip `$0` rows. Positive `Amount` = purchase (`expense`); negative = refund/payment (`income`). Use Merchant as `name`, Description as `notes`.
- **Amazon orders:** Many rows share an `Order ID` and repeat `Total Amount`. Collapse to **one Sure transaction per order** using order total. Skip `Cancelled` / zero totals. Split mixed orders only if the user asks.
- **Venmo:** Header row contains `Datetime` and `Note` (not row 0). `Amount (total)` sign is cash flow (`+` received / `-` sent). Filter `filteredRows`. Default window is the **previous calendar month**.

### Sure API

Public UI is Authentik-fronted (`https://sure.tyler.cloud`); `/api` is **not** skipped. Call the LAN origin from Cabinet `sure.base_url` (default `http://192.168.1.101:3006`).

```
GET/POST  /api/v1/accounts
GET       /api/v1/categories?per_page=100   (paginate; ~70 categories)
GET       /api/v1/transactions?account_id=&start_date=&end_date=&page=&per_page=100
POST      /api/v1/transactions
PUT       /api/v1/transactions/{id}
DELETE    /api/v1/transactions/{id}
GET       /api/v1/balance_sheet             (live net worth / assets / liabilities)
GET       /api/v1/balances?account_id=&start_date=&end_date=&page=&per_page=100
```

Header: `X-Api-Key` (Cabinet `sure.api_key`). JSON body:

```json
{ "transaction": {
    "account_id": "<uuid>",
    "date": "YYYY-MM-DD",
    "amount": 12.34,
    "nature": "expense",
    "name": "…",
    "notes": "…",
    "category_id": "<uuid>",
    "external_id": "stable-id",
    "source": "venmo|everbank|robinhood_cc|amazon",
    "user_modified": true
}}
```

- `amount` is always **positive**. `nature` is `expense`/`outflow` (money leaving the account) or `income`/`inflow` (money entering). Credit-card purchases are `expense`; card payments and refunds are `income`.
- List responses use `signed_amount_cents`: **income positive, expense negative**. Match CSV rows to existing txns on **date + signed cents**, then name/merchant. Skip Sure names like `Manual balance update`.
- Do **not** stamp category `Other`. Leave unmatched notes uncategorized. Do **not** overwrite a more specific Sure category with a weaker keyword match.
- Re-runs: skip when date/amount (and `external_id` if present) already exist. Prefer a stable `external_id` (Venmo `ID`, Amazon order ID, hash of date+description+amount).
- Paginate everything (`per_page` max 100).

Compose / Rails: Sure host in `sure.base_url` (`~/git/docker/sure.am`). Category tree edits that the API cannot do (rename, merge, destroy) go through `docker exec sure-web bin/rails runner '…'`.

## Credentials (Cabinet)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tylerjwoodfin/tools](https://github.com/tylerjwoodfin/tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

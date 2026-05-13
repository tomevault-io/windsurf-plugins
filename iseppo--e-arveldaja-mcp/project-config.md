---
trigger: always_on
description: TypeScript MCP server for the Estonian e-arveldaja (RIK e-Financials) REST API.
---

# e-arveldaja MCP Server

TypeScript MCP server for the Estonian e-arveldaja (RIK e-Financials) REST API.
111 tools, 15 workflow prompts, 12 resources across 12 modules. Supports multiple companies/accounts.

## Quick Start

```bash
npm run build          # tsc -> dist/
npm run start          # node dist/index.js (stdio transport)
npm run dev            # tsx src/index.ts (development)
```

## Credentials

All `apikey*.txt` files are scanned from the working directory (where the server is launched). Multiple files = multiple connections (companies).

Credentials are loaded in this priority order (see `src/config.ts`):

1. **`EARVELDAJA_API_KEY_FILE`** env var pointing to a specific file
2. **Environment variables**: `EARVELDAJA_API_KEY_ID`, `EARVELDAJA_API_PUBLIC_VALUE`, `EARVELDAJA_API_PASSWORD`
3. **`.env` files** — local working directory first, then global config directory (`~/.config/e-arveldaja-mcp`)
4. **`apikey*.txt` files** — scanned from the working directory (import source, not long-term store)

The `apikey.txt` format:
```
ApiKey ID: <key_id>
ApiKey public value: <public_value>
Password: <password>
```

Set `EARVELDAJA_SERVER=demo` for the demo API (default: `live`).

### Multi-account (multiple companies)

Place multiple `apikey*.txt` files (e.g. `apikey.txt`, `apikey (1).txt`) next to the project.
Use `list_connections` to see all available accounts and `switch_connection` to switch between them.
Switching clears all cached data to prevent cross-company data leaks.

**NEVER commit `.env` or `apikey.txt` to git.** The `.gitignore` is configured to exclude them.

## Authentication

HMAC-SHA-384 signing (`src/auth.ts`):
- Message: `{keyId}:{utcTime}:{urlPath}`
- Signature: `BASE64(HMAC-SHA-384(message, password))`
- Headers: `X-AUTH-KEY: {publicValue}:{signature}`, `X-AUTH-QUERYTIME: {utcTime}`
- Signing uses the URL path only (no query params)

## API Endpoints (RIK e-Financials v1)

OpenAPI spec: `GET /openapi.yaml` on the API server. HTML docs: `/api.html`.

### Action endpoints
- **Confirm/Register**: `PATCH /{entity}/{id}/register` (not `/confirm`)
- **Invalidate**: `PATCH /{entity}/{id}/invalidate`
- **Reactivate**: `PATCH /clients|products/{id}/reactivate`
- **Deactivate**: `PATCH /clients|products/{id}/deactivate`
- **Deliver sale invoice**: `PATCH /sale_invoices/{id}/deliver` (not `/send_einvoice`)

### Document endpoints
- **User-uploaded docs**: `GET/PUT/DELETE /{entity}/{id}/document_user` (PUT to upload, not POST)
- **System-generated sale invoice PDF**: `GET /sale_invoices/{id}/pdf_system`

### Transaction registration
- Body is a **top-level JSON array** of `TransactionsDistribution` objects (not wrapped in `{items: [...]}`)
- Each distribution: `{related_table, amount, related_id?, related_sub_id?}`
- `related_table` values: `"accounts"` (book to a GL account), `"purchase_invoices"`, `"sale_invoices"`
- Example (purchase invoice): `PATCH /transactions/{id}/register` with body `[{"related_table":"purchase_invoices","related_id":123,"amount":59.94}]`
- Example (account with dimensions): `[{"related_table":"accounts","related_id":1360,"related_sub_id":12637323,"amount":1620.70}]`
- **`related_sub_id` is REQUIRED when `related_table="accounts"` and the account has dimensions.** Pass the dimension ID there. Without it, the API rejects with `"Entry cannot be made directly to the account <code> since it has dimensions"`. Common case: account 1360 "Arveldused aruandvate isikutega" with one sub-account per reporting person.
- **Do NOT pass the dimension ID into `related_id`.** That makes the API try to interpret it as an account ID and produces confusing errors like `"Dimension ID=<truncated> not found"`.
- The dimension ID is the integer `id` value returned by `list_account_dimensions` (NOT a sub-account code or label). `related_id` is the integer account ID, `related_sub_id` is the integer dimension ID — both are the database IDs, not the human-readable account/dimension codes.
- **Card payments often have `clients_id: null`** — confirmation fails with "buyer or supplier is missing". `TransactionsApi.confirm()` auto-fixes this by looking up the client from the linked invoice.

### Inline confirmation policy
- When a workflow leaves behind `PROJECT` (unconfirmed) transactions, journals, or `needs_review` items and the agent has the IDs/amounts/counterparties loaded, **always offer inline confirmation** via `confirm_transaction` / `reconcile_inter_account_transfers` / `update_transaction` (for `bank_ref_number` enrichment) / `delete_transaction`. Ask the user yes/no for each item.
- **Never** close a workflow with "these need your manual confirmation in e-arveldaja" / "tee see e-arveldaja UI-s käsitsi" as the default. That is a last-resort fallback only when (a) no MCP tool can perform the action, AND (b) the exact API error has already been shown to the user with what was tried.
- If the API rejects an inline attempt, show the raw API error and the exact body that was sent before suggesting manual UI fallback.

### Transaction type field
- **All bank transactions are `type: "C"`** regardless of direction (both CAMT-imported and API-created)
- The `type` field is cosmetic — it does **not** affect accounting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iseppo/e-arveldaja-mcp](https://github.com/iseppo/e-arveldaja-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

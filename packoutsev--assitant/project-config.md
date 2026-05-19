---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Assitant** — a Xactimate expert assistant app for 1-800-Packouts (packout/contents restoration company). Repository: https://github.com/packoutsev/Assitant.git

## Business Context

- **Company**: 1-800-Packouts of the East Valley (Encanto Builders DBA)
- **Owner**: Matthew Roumain, acquired the business **April 15, 2025** — anything before that date was previous ownership
- **What they do**: Contents packout, cleaning, storage, and pack-back for insurance restoration jobs
- **Xactimate**: Industry-standard estimating software used for all job estimates (initial, supplement, final)
- **Encircle**: Photo documentation/inventory app — Encircle reports directly correlate to Xactimate estimates (same customers/jobs)

## Xactimate Data Analysis (338 estimates, $4.53M total RCV)

Key findings from initial analysis of exported Excel estimates:
- **Labor is 38% of all revenue** — packing labor at ~$59/hr and supervisor at ~$81/hr dominate
- **Storage is #2 revenue driver** — ~$407K across 161 estimates
- **Huge unit cost variation** on same line items (e.g., moving vans range $0-$600/day, storage $145-$425/month) — pricing inconsistency
- **Med box high-density packing** ($320K) is a bigger revenue item than most realize
- **Cleaning is underrepresented** — only 34 of 243 packout estimates include cleaning scope
- **Zero depreciation** across all estimates — everything at full RCV
- **5.5% of line items have zero quantity** (template placeholders never filled in)
- Estimate values range from $0 to $167K, median ~$10K

## Data Locations

- **Xactimate Excel exports**: `C:\Users\matth\Downloads\Spreadsheets\Xactimate Estimates\` (338 files)
- **Other spreadsheets**: `C:\Users\matth\Downloads\Spreadsheets\`
- **Downloads organized into**: PDFs/, Images/, Spreadsheets/, Documents/, Videos/, Archives/, Installers/, Emails/, ESX/
- **Xactimate PDF reports & Encircle photo reports**: Available via Google Drive MCP server

## MCP Servers

Six MCP servers provide tool access to external systems. Five are deployed to **Google Cloud Run** (project `packouts-assistant-1800`, region `us-central1`) for use as custom connectors in claude.ai. Each has two entry points: `index.js` (stdio for Claude Code) and `server.js` (Streamable HTTP for Cloud Run).

### Cloud Run Deployments

| Server | URL | Tools |
|--------|-----|-------|
| **mcp-encircle** | `https://mcp-encircle-326811155221.us-central1.run.app/mcp` | Claims, photos, rooms, moisture readings, media, equipment, notes |
| **mcp-qbo** | `https://mcp-qbo-326811155221.us-central1.run.app/mcp` | Invoices, A/R aging, P&L, balance sheet, customers, QBO SQL |
| **mcp-xcelerate** | `https://xceleratewebhook-326811155221.us-central1.run.app/mcp` | Jobs, schedule, notes, status (Zapier webhook → Firestore + MCP) |
| **mcp-gchat** | `https://mcp-gchat-326811155221.us-central1.run.app/mcp` | Google Chat spaces, messages, threads, search, send messages (read+write) |
| **mcp-gsheets** | `https://mcp-gsheets-326811155221.us-central1.run.app/mcp` | Google Sheets read/write — open spreadsheet, list tabs, read/write ranges, append rows |

**Transport**: Streamable HTTP (`POST /mcp`) via MCP SDK v1.26.0 `StreamableHTTPServerTransport`. Legacy SSE not supported — claude.ai requires Streamable HTTP for custom connectors.

**Auth**: No bearer token required (claude.ai custom connectors don't support simple bearer tokens — only OAuth or no-auth). API credentials (Encircle API token, QBO OAuth, Google Chat/Sheets OAuth) are server-side env vars on Cloud Run. Google Chat and Sheets tokens stored in GCS bucket `packouts-gchat-tokens` (different file paths: `tokens.json` for Chat, `gsheets-tokens.json` for Sheets).

**Deploying updates**: From each `mcp-*/` directory:
```bash
gcloud run deploy <service-name> --source . --region us-central1 --no-invoker-iam-check --quiet
```
The `--no-invoker-iam-check` flag bypasses the GCP org policy that blocks `allUsers` IAM binding.

### Local-Only MCP Servers

| Server | Location | Notes |
|--------|----------|-------|
| **mcp-gdrive** | `C:\Users\matth\mcp-gdrive-setup\` | Google Drive file search (patched for Windows). If auth expires, re-run: `node node_modules\@modelcontextprotocol\server-gdrive\dist\index.js auth` |

### Google Cloud CLI

- Installed at `C:\Users\matth\google-cloud-sdk\` (v557.0.0)
- Authenticated as `matt@encantobuilders.com`
- Added to Windows PATH permanently

## Estimator Tool (Built)

The back half of the pipeline is built and validated:
- **`estimator/photo_analyzer.py`** — Room-level TAG/box counting (lookup tables + per-room overrides)
- **`estimator/pricing_engine.py`** — Xactimate line item generation with CPS codes
- **`estimator/generate_estimate.py`** — Full 5-phase estimate pipeline (Packout → Storage → Pack back)
- **`estimator/cartage_calculator.py`** — Handling labor hours from drive time, crew, truck loads
- **Backtest results**: 8.6% MAPE with photo overrides, 13.6% overall (target <20%)
- **Key insight**: Pricing engine is accurate — the bottleneck is input quality (getting accurate per-room TAG/box counts)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [packoutsev/Assitant](https://github.com/packoutsev/Assitant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->

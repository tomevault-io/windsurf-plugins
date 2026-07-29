---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Official Postmark MCP (Model Context Protocol) server that enables AI assistants (Claude, Cursor, etc.) to interact with the Postmark email API. Published as `@activecampaign/postmark-mcp` on NPM.

## Commands

- **Run server:** `npm start` (runs `node index.js`)
- **Debug with MCP Inspector:** `npm run inspector` (launches `@modelcontextprotocol/inspector`)
- **Smoke test (read-only):** `npm run smoke` — runs `smoke-test.mjs`, which is created from [smoke-test.example.mjs](smoke-test.example.mjs). Spawns the server over stdio and exercises every read-only tool against the live Postmark account configured in `.env`. Does not send mail or mutate state.
- **Smoke test (mutating):** `node smoke-test-mutating.mjs` — runs full create→edit→delete lifecycles for templates (including layout binding), webhooks, and suppressions, plus real email sends between two configured verified addresses. Cleans up after itself. Created from [smoke-test-mutating.example.mjs](smoke-test-mutating.example.mjs); edit `SENDER` and `RECIPIENT` before running. The script refuses to run with the placeholder values still in place.
- **`smoke-test.mjs` and `smoke-test-mutating.mjs` are gitignored.** Copy from the `*.example.mjs` counterparts; this keeps personal verified-sender addresses out of the repo.
- **No linter is configured.**

## Architecture

This is a single-file MCP server (`index.js`) using ES modules (`"type": "module"`). The structure within `index.js`:

1. **Module-level formatting helpers** — `fmtInt`, `fmtPct`, `fmtPlatformUsage`, `fmtTopBreakdown`, `fmtDeliverySummary`, `fmtStatResponse`. Used by `getDeliveryStats` to render polished per-stat output. Keep these pure / dependency-free.
2. **`initializeServices()`** — Validates required env vars, verifies connectivity via a `postmarkRequest('/server')` call, and instantiates `McpServer` from `@modelcontextprotocol/sdk`
3. **`registerTools(server)`** — Registers all 24 MCP tools with Zod schemas for input validation. Each tool wraps a `postmarkRequest` call and returns formatted text responses
4. **`main()`** — Orchestrates initialization, tool registration, and connects to `StdioServerTransport`

Communication uses stdio transport (stdin/stdout), not HTTP. All `console.error` calls are diagnostic logging (stdout is reserved for MCP protocol).

## MCP Tools Registered

The server exposes 24 tools across these categories:
- **Email:** sendEmail, sendEmailWithTemplate, sendBatch, sendBatchWithTemplate
- **Templates:** listTemplates, getTemplate, createTemplate, editTemplate, deleteTemplate, validateTemplate
- **Messages:** searchOutboundMessages, getMessageDetails
- **Diagnostics:** diagnoseDelivery
- **Bounces:** searchBounces, getBounceDump, activateBounce
- **Suppressions:** listSuppressions, createSuppressions, deleteSuppressions
- **Stats & Server:** getDeliveryStats, getServerInfo
- **Webhooks:** listWebhooks, createWebhook, deleteWebhook

`sendBatch` and `sendBatchWithTemplate` wrap Postmark's *synchronous* batch endpoints (`/email/batch` and `/email/batchWithTemplates` respectively), each accepting up to 500 messages. Both share a `formatBatchResults` helper defined inside `registerTools` that splits the response by `ErrorCode` (0 = success, non-zero = failure) and renders a summary with capped success/failure lists. `sendBatchWithTemplate` accepts a top-level `from` / `tag` that's overridable per recipient — keep that override semantics if extending. Note: these are Postmark's *synchronous* batch endpoints — the API call returns per-message results in the response. Postmark's separate *asynchronous* bulk email API (`/email/bulk` — submit a job, get a request ID, poll `GET /email/bulk/{id}` for status; no message count cap, 50 MB payload limit; subject to approval) is a parallel capability not covered in v2.0. The two are parallel APIs in Postmark's docs, not one replacing the other; bulk is tracked as a v2.1 follow-up. If anyone asks "where's the bulk send tool," that's why — and adding it is more than a quick wrap because the async submit-and-poll pattern needs two tools (`submitBulk` + `getBulkStatus`).

`diagnoseDelivery` is a **composite tool** — it does not mirror a single Postmark endpoint. Instead it runs `getOutboundMessages` (or `getOutboundMessageDetails`), `getSuppressions`, and `getBounces` in parallel for the given recipient, then synthesizes a plain-English recommendation. When adding new diagnostic tools, follow the same pattern: parallelize independent lookups via `Promise.all`, swallow individual failures with `.catch(() => fallback)` so one 404 doesn't sink the whole diagnosis, and end with a `Recommended action` block that interprets the data.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ActiveCampaign/postmark-mcp](https://github.com/ActiveCampaign/postmark-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

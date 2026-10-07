---
trigger: always_on
description: MCP server for OurFamilyWizard (OFW). Reads/writes messages, calendar, expenses, and journal; backs message tools with a local SQLite cache. stdio transport.
---

# ofw-mcp

MCP server for OurFamilyWizard (OFW). Reads/writes messages, calendar, expenses, and journal; backs message tools with a local SQLite cache. stdio transport.

## Commands

```bash
npm run build        # tsc → dist/, then esbuild bundle → dist/bundle.js
npm test             # tsc typecheck + vitest run (all tests)
npm run test:watch   # vitest in watch mode
npm run dev          # node --env-file=.env dist/index.js (requires built dist)
```

`dist/` is gitignored — it is produced at build/release time and shipped in the npm package (`package.json` `files`).

## Session cache

`src/session-cache.ts` caches the bearer token at
`$MCP_DATA_DIR/.ofw-mcp/session.json` (0600, `OFW_SESSION_FILE` overrides) via
`createFileStatePersistence` + `resolveStateFile` (`@chrischall/mcp-utils/session`).

OFW has no refresh grant — renewing IS logging in — so `mint()` serves as both
`initial` and `refresh` on the `TokenManager`, and `isRefreshRevoked: () => false`
turns off the library's re-mint-after-a-failed-refresh recovery, which here would
just repeat the call that failed. The token lasts six hours, so on a scale-to-zero
host the cache turns most cold starts into zero-cost ones.

The cache is **disabled** in three cases, each deliberate: `OFW_SESSION_CACHE=false`;
an injected `resolveAuth` (the per-user hosted path — this registration declares no
`identity.perUserChild`, so one process can serve several people and a shared file
would hand one user's session to the next); and no env credentials (the fetchproxy
path authenticates from a browser tab, with nothing stable to bind a record to).

The record is `boundTo` a salted digest of username+password, so rotating either
discards it; neither value is written. A failed write is reported to stderr and is
NOT fatal — OFW tokens are re-mintable from the environment.

`tests/_setup.ts` forces the cache off and pins its path into a temp dir for the
whole suite, so no test can reach the developer's real `~/.ofw-mcp`.

## Architecture

```
src/
  index.ts          MCP server entry — SQLite-warning shim, then runMcp() from @chrischall/mcp-utils (builds McpServer, applies registrars with client as deps, prints banner, wires shutdown + stdio transport)
  protocol.ts       Wire-level constants (BASE_URL, OFW_PROTOCOL_HEADERS, token TTL) + assertOfwUrl() egress allowlist. Leaf module to break the client→auth→auth-password import cycle
  client.ts         OFWClient (Bearer token, 401/429 retry, JSON + binary). Delegates auth to ./auth.ts
  auth.ts           resolveAuth(): three-path priority (env vars → fetchproxy fallback → error). Template for sibling MCPs
  auth-password.ts  loginWithPassword(): legacy OFW Spring Security form login (kept as own module so auth.ts can mock it cleanly)
  config.ts         env-driven cache dir + sha256(OFW_CACHE_IDENTITY|OFW_USERNAME|"_default") DB path + attachments dir
  cache.ts          node:sqlite cache (messages, drafts, draft_lineage, attachments, sync_state, meta) with typed CRUD + findLatestReplyTip
  sync.ts           resolveFolderIds + syncMessageFolder/syncDrafts/syncAll + attachment-meta fetch
  extract/          format-agnostic content extraction (no deps, runtime-portable)
    zip.ts          minimal ZIP reader (DecompressionStream, not node:zlib)
    xml.ts          scanning XML reader (entity decode, full-tag-name matching)
    ooxml.ts        shared part-path + relationship resolution
    spreadsheet.ts  .xlsx/.xlsm → per-sheet CSV; .csv/.tsv
    document.ts     .docx → text with headings/lists/tables
    presentation.ts .pptx → per-slide text + speaker notes
    pdf.ts          .pdf → text layer (or an explicit "needs OCR" verdict)
    index.ts        extractAttachment(): format detection, part selection, char budget
  tools/
    _shared.ts      recipient mapping, response helpers, path expansion
    delivery.ts     the attachment delivery ladder (image → extracted → raw bytes)
    freshness.ts    buildFreshness() — the `freshness` block every read tool returns (source/asOf/ageSeconds/staleness/warning)
    pagination.ts   paging state that survives a lossy reader: pageState/offsetState/readUpstreamPaging/withPaginationFirst
    project.ts      the `view` projection — compactMessage/compactDraft/viewMessages/viewDrafts/viewOne, MESSAGE_VIEWS
    lifecycle.ts    "is this entity still what I think it is?" — classifyState/probeIds/resolveDraftKey/newDraftKey
    user.ts         ofw_get_profile, ofw_get_notifications
    messages.ts     folders, list, get, send, drafts, get_unread_sent, upload/download_attachment, sync_messages, check_freshness, status
    calendar.ts     list/create/update/delete events
    expenses.ts     totals, list, create
    journal.ts      list, create entries
tests/              mirrors src/; mocks OFWClient.request via vi.spyOn; cache tests use OFW_CACHE_DIR + tmp dir
```

Tool files use `server.registerTool(name, schema, handler)` and export `registerXTools(server: McpServer, client: OFWClient)`. `index.ts` passes those registrars to `runMcp({ tools: [...], deps: client })`, which calls each as `registerXTools(server, client)`.

## Environment

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrischall/ofw-mcp](https://github.com/chrischall/ofw-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

---
trigger: always_on
description: A Chrome extension (Manifest V3) that captures web page content and sends it to a local HTTP server, which stores it in a ClipBrain knowledge base via the `gbrain` CLI.
---

# ClipBrain — Chrome Extension + HTTP Server

## What it does

A Chrome extension (Manifest V3) that captures web page content and sends it to a local HTTP server, which stores it in a ClipBrain knowledge base via the `gbrain` CLI.

This project uses the installed `gbrain` CLI. `./setup.sh` installs it globally with Bun if it is missing.

## Architecture

### Chrome Extension

MV3 service workers do NOT have DOM access, so the work is split:

- **content-script.js** — Injected into the active tab on demand. Has DOM access. On YouTube video pages, extracts the video ID, title, channel name, and **transcript** (from the page's inline `captionTracks` data, fetched same-origin so YouTube cookies are included). Falls back to reading the transcript panel DOM if the script extraction fails. Sends a `youtube-capture` message with the transcript segments already extracted. On all other pages, runs Mozilla's Readability.js to extract the article text. Sends the extracted data back to the service worker via `chrome.runtime.sendMessage`.
- **kindle-content-script.js** — Auto-injected on `read.amazon.com/notebook*`. Parses Kindle highlights/notes from the Notebook page and sends them to the service worker as `kindle-import` messages. Shows a floating "Import to ClipBrain" button in the bottom-right corner.
- **gmail-content-script.js** — Injected into Gmail after the user grants the optional `mail.google.com` permission from the popup. Detects open emails, extracts subject, sender, date, and clean body text (stripping Gmail UI chrome, signatures, tracking pixels). Shows a floating "Clip to ClipBrain" button when an email is open. Supports both button click and Cmd+Shift+S. Sends `gmail-capture` messages to the service worker. Handles threaded conversations (joins all messages). Uses `gmail://` URL scheme for dedup. Slugs stored as `email/{from-slug}/{subject-slug}`.
- **service-worker.js** — Background service worker. Listens for the `capture-page` keyboard command, injects the content script, receives extracted content (types `captured`, `kindle-import`, and `youtube-capture`), and POSTs it to the local HTTP server. YouTube captures go to `/api/capture-youtube`. Manages an offline queue in `chrome.storage.local` and flushes it via `chrome.alarms`.
- **toast.js** — Injected into the page to show a brief success/failure notification.
- **lib/readability.js** — Vendored copy of Mozilla Readability.js (from `@mozilla/readability` npm package).

### HTTP Server (server.ts)

A standalone Bun HTTP server that:

- Listens on port 19285 (configurable via `--port` or `GBRAIN_CAPTURE_PORT` env)
- Receives POST /api/capture with `{ url, title, content, selection? }`
- Receives POST /api/capture-youtube with `{ url, videoId, title, channel, transcript }` — transcript segments are extracted client-side by the content script (same-origin fetch with YouTube cookies); the server formats them with timestamps and saves as `youtube/{channel-slug}/{title-slug}`
- Canonicalizes the URL, generates a slug, builds markdown with frontmatter
- Handles `kindle://` URLs specially: generates slugs as `kindle/{author}/{title}` from the title field
- Handles `gmail://` URLs: generates slugs as `email/{from-slug}/{subject-slug}`
- Calls `gbrain put <slug>` via CLI (content piped via stdin)
- Returns 202 immediately (fire-and-forget)
- Handles CORS for chrome-extension:// origins
- Appends every capture to `.captures.jsonl` (append-only, gitignored) and exposes `GET /api/digest?since=ISO|days=N` — returns captures since a date grouped by type (kindle/web/youtube/email/pdf) plus a Slack-friendly markdown summary. Kindle entries include `newHighlights` (delta vs previous capture).
- Exposes `GET /api/context-pack?q=...&limit=...` — returns structured sources plus agent-ready markdown with `[S#]` citations, retrieval snippets, summaries, claims, quotes, entities, open questions, and actions.

The server resolves the gbrain binary in this order: `GBRAIN_BIN` env var, then `gbrain` on `PATH`.

### MCP Bridge (clipbrain-mcp.ts)

ClipBrain registers a small MCP server alongside `gbrain serve`:

- `gbrain` MCP keeps the broad knowledge-engine tools (`query`, `search`, `get_page`, graph tools, etc.)
- `clipbrain` MCP exposes `context_pack`, which calls the local HTTP server and returns the same cited handoff used by `/api/context-pack`

The ClipBrain MCP server is local-only by default and reads `CLIPBRAIN_SERVER_URL` from the host environment, falling back to `http://127.0.0.1:19285`.

### Post-Processing (post-process.ts)

After each capture, if `OPENAI_API_KEY` is set, the server runs AI post-processing in the background:
- Generates a 2-3 sentence summary
- Creates 3-5 semantic tags
- Finds connections to existing content in the knowledge base
- Extracts knowledge atoms: claims, quotes, entities, open questions, and actions
- Enriches the markdown with `## Summary`, `## Why It Matters`, `## Knowledge Atoms`, and `## Related` sections
- If `gbrain put` rejects an enriched page because the embedding context is too
  large, stores the parent as a compact Knowledge Compiler page and writes the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentpilled/ClipBrain](https://github.com/agentpilled/ClipBrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->

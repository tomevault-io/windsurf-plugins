---
trigger: always_on
description: xTap is a browser extension (Chrome + Firefox) that passively captures tweets from X/Twitter by intercepting GraphQL API responses the browser already receives. No scraping, no extra requests — just structured JSONL output of what the user sees.
---

# AGENTS.md - xTap

## What This Is

xTap is a browser extension (Chrome + Firefox) that passively captures tweets from X/Twitter by intercepting GraphQL API responses the browser already receives. No scraping, no extra requests — just structured JSONL output of what the user sees.

**Repo:** github.com/mkubicek/xTap
**License:** MIT (public repo)

## Architecture

```
content-main.js (MAIN world)
  │  Patches fetch() + XHR.open() to intercept GraphQL responses
  │  Emits CustomEvent with random per-page name
  ▼
content-bridge.js (ISOLATED world)
  │  Reads event name from <meta> tag, listens, relays
  │  Removes <meta> immediately after reading
  ▼
background.js (Service Worker, ES module)
  │  Parses tweet data via lib/tweet-parser.js
  │  Deduplicates (Set of seen IDs, max 50k; session storage in dev, local in prod)
  │  Batches (50 tweets or 30–45s jittered flush)
  │  Debug logging: intercepts console.log/warn/error, sends to host
  │  Transport: HTTP daemon only (reprobes on failure with 30s cooldown)
  ▼
┌─── HTTP transport ─────────────────────────────────────────────┐
│ xtap_daemon.py (127.0.0.1:17381)                               │
│   Managed by launchd (macOS), systemd (Linux), Scheduled Task  │
│   (Windows). Bearer token auth from ~/.xtap/secret             │
│   Endpoints: GET /status, POST /tweets, /log, /test-path,     │
│   /check-ytdlp, /download-video, /download-status             │
└────────────────────────────────────────────────────────────────┘
┌─── Native messaging (token bootstrap only) ───────────────────┐
│ xtap_host.py (Python, stdio)                                   │
│   Browser native messaging protocol (Chrome/Firefox)           │
│   Serves GET_TOKEN to bootstrap HTTP transport                 │
│   Crashes logged to ~/.xtap/host-error.log                    │
└────────────────────────────────────────────────────────────────┘
  │  xtap_daemon.py uses shared logic from xtap_core.py
  ▼
tweets-YYYY-MM-DD.jsonl  (daily rotation)
debug-YYYY-MM-DD.log     (when debug logging enabled)
```

### Key Design Decisions

- **Two content scripts (MAIN + ISOLATED):** MV3 requires this split. MAIN world can patch browser APIs but can't use chrome.runtime. ISOLATED world bridges the gap.
- **Random event channel:** The CustomEvent name is generated per page load (`'_' + Math.random().toString(36).slice(2)`) and passed via a `<meta>` tag that's immediately removed. Avoids predictable DOM markers.
- **HTTP-only transport:** All data flows through the HTTP daemon (`xtap_daemon.py`), managed by launchd (macOS), systemd (Linux), or Scheduled Task (Windows). On macOS, it runs outside browser TCC sandboxes, allowing writes to protected paths. If the daemon goes down, tweets are buffered in memory and the extension reprobes every 30 seconds until the daemon recovers.
- **Token bootstrap via native messaging:** On first run, the extension connects to `xtap_host.py` via native messaging to request `GET_TOKEN` (reads `~/.xtap/secret`). The token is cached in `chrome.storage.local` for subsequent HTTP requests. The native host handles nothing else — all data goes through HTTP.
- **Shared core logic:** `xtap_core.py` contains all file I/O logic (load seen IDs, resolve output dir, write tweets/logs, test path), used by `xtap_daemon.py`.
- **Environment detection:** `isDevMode = !chrome.runtime.getManifest().update_url` — packed CWS extensions have `update_url`, unpacked don't. Used to switch seenIds storage between session (dev) and local (production).
- **Volatile dev cache:** In dev mode (unpacked), `seenIds` is stored in `chrome.storage.session`, which clears on extension reload. This eliminates the need to manually clear storage during development. Production behavior is unchanged (persisted to `chrome.storage.local`).
- **Dedup in service worker:** Multiple tabs feed the same service worker. `seenIds` Set (max 50,000, FIFO eviction) prevents duplicates. In production, persisted to `chrome.storage.local` across sessions; in dev mode, uses volatile `chrome.storage.session`. Both host and daemon also load seen IDs from existing JSONL files on startup.
- **Jittered flush:** Batch flush uses `setTimeout` with randomized interval (30s base + up to 50% jitter = 30–45s), re-randomized each cycle. Avoids clockwork-regular patterns.
- **Path validation:** When the user sets a custom output directory, the service worker sends a `TEST_PATH` request to the HTTP daemon, which attempts `makedirs` + write/delete of a temp file before accepting the path.
- **Error resilience:** The native host logs crashes to `~/.xtap/host-error.log` with Python version and traceback. The HTTP daemon returns error status codes and logs startup diagnostics (Python version, output dir, token status). When the daemon is unreachable, the extension shows a red "!" badge and buffers tweets until the next successful reprobe (30s cooldown). The popup auto-refreshes every 2 seconds to reflect transport state changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mkubicek/xTap](https://github.com/mkubicek/xTap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->

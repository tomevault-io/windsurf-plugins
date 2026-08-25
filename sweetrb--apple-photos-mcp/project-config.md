---
trigger: always_on
description: This file provides guidance for AI agents (Claude, etc.) when using this MCP server.
---

# CLAUDE.md - Apple Photos MCP Server

This file provides guidance for AI agents (Claude, etc.) when using this MCP server.

## Overview

This MCP server gives AI assistants access to the macOS Apple Photos library
via [osxphotos](https://github.com/RhetTbull/osxphotos) — **read-only by
default**. All operations are **local** — nothing leaves the user's machine.
You can query the library (including by GPS radius, aesthetic score, and
OCR-detected text), inspect individual photos (singly or in batches, including
EXIF camera data, ML score/detected text, shared-album comments/likes, and
burst siblings), read the **live Photos.app selection**, **see** photos inline
via thumbnails, find exact-duplicate groups, browse the library's structure
(albums, folders, keywords, persons), and **export** copies of photos to a
directory.

**Write tools exist but are gated:** `create-album`, `add-to-album`,
`remove-from-album`, `set-photo-metadata`, `set-keywords`, `set-photo-date`,
and `import-photos` only work when the user has set
`APPLE_PHOTOS_MCP_ENABLE_WRITES=1` (env or config.json + server restart).
Until then every write call returns a clear opt-in error — **run `doctor`
first** when writes matter: its `writes` check reports the gate state. Even
with writes enabled, **nothing can delete a photo** (see "Write workflow"
below).

## Related Documentation

- **[docs/FULL-DISK-ACCESS.md](./docs/FULL-DISK-ACCESS.md)** — why Full Disk
  Access is required, how to grant it, and how to verify it. Required reading
  when tools fail with permission errors.
- **[docs/LIMITATIONS.md](./docs/LIMITATIONS.md)** — what the server can and
  can't do (read-only, iCloud export caveats, face/album behavior, library lag).
- **[docs/QUERY-GUIDE.md](./docs/QUERY-GUIDE.md)** — `query` filter syntax in
  detail: accepted date forms, AND/OR combination semantics, exact-vs-substring
  matching, result ordering, and what is *not* filterable.
- **[docs/WRITE-BACKEND.md](./docs/WRITE-BACKEND.md)** — why reads use osxphotos
  (direct DB) and writes use photoscript/AppleScript, and why that split can't be
  collapsed (no safe DB writes; osxphotos's own writes *are* AppleScript; PhotoKit
  needs an app bundle). Read before proposing to "eliminate AppleScript" in writes.

## First-run requirements

Before any tool works, two things must be in place:

1. **Python sidecar installed — self-installing.** The server shells out to
   osxphotos (Python). For most users this needs **no manual step**: the venv at
   `./venv` is built automatically on the first tool call (a one-time setup that
   can take ~a minute; progress logs to stderr), then the call proceeds. The venv
   is also picked up without a restart once it exists, and rebuilt automatically
   if a package update changes its requirements. Pre-warm it with `pnpm run setup`
   to skip the first-call delay. Auto-setup needs Python 3, `pip`, and network
   access; it can be disabled with `APPLE_PHOTOS_MCP_NO_AUTO_SETUP=1` (then you
   must run `pnpm run setup` or `pip3 install osxphotos` yourself).
2. **Full Disk Access granted** to the host app (Claude/Terminal/iTerm/VS Code),
   then the host app **fully restarted**. The Photos library database is in a
   protected directory; without FDA, *every* tool fails. See
   [docs/FULL-DISK-ACCESS.md](./docs/FULL-DISK-ACCESS.md).

Run **`health-check`** first when in doubt — it confirms both at once (osxphotos
present + library openable). When something is actually broken, reach for
**`doctor`**: it's the richest diagnostic, checking the Python interpreter
(path + version — warns below the required 3.11), osxphotos install, the
sidecar execution mode (persistent vs one-shot fallback), the write-tools gate
(enabled/disabled + backend readiness), library readability, and Full Disk
Access separately and reporting each as ok / warn / fail with an actionable
message — so it pinpoints *which* of the first-run requirements is missing.

## Performance model: cold vs warm calls

The Python sidecar runs as a **persistent process**: the first tool call pays a
one-time cost (python start + osxphotos import + a full parse of the library
database — roughly ~4 s on a ~30k-photo library, more on bigger ones), and
subsequent calls reuse the resident parsed library and complete in
**milliseconds**. The sidecar re-checks the library's modification time before
every request, so results are never stale — a changed library (import, edit,
album rename) triggers an automatic re-parse on the next call. After 5 idle
minutes (`APPLE_PHOTOS_MCP_SIDECAR_IDLE_MS`) the process is killed to free
memory and the next call pays the cold cost again. Practical upshot: don't
avoid follow-up calls for cost reasons — a `query` → `get-photo` → `export`
chain costs the parse once, not three times. Batch `export`s report per-photo
MCP progress notifications when the request carries a `progressToken`.

**Don't rely on the final `done === total` progress notification.** Treat the
per-photo notifications as the progress signal and the **tool result** (its
`exportedCount` / `skippedCount`) as the completion signal. An MCP client
deletes a request's progress handler the moment the response arrives and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sweetrb/apple-photos-mcp](https://github.com/sweetrb/apple-photos-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->

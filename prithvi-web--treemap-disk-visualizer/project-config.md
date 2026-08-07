---
trigger: always_on
description: TreeMap is a local, privacy-preserving disk-space visualizer (Node 20 + Express).
---

# TreeMap for agents

TreeMap is a local, privacy-preserving disk-space visualizer (Node 20 + Express).
This file documents how an automated agent should drive it — the workflows and,
above all, the safety model. Machine-readable equivalents:
`GET /api/capabilities` (compact manifest) and `GET /api/openapi.json` (full
OpenAPI 3 spec).

## Two ways in

- **HTTP API** — start the server (`npm start`, default `http://127.0.0.1:4280`)
  and call `/api/*`. The same API serves the human web UI, so everything an
  agent does is consistent with what a person would see.
- **MCP** — `npm run mcp` starts a stdio Model Context Protocol server
  (for Claude Desktop and similar clients) exposing: `scan_path`,
  `get_largest`, `find_duplicates`, `cleanup_suggestions`, `forecast`,
  `compare_scans`, `offload`, `trash_paths`. The tools call the exact same
  internals as the HTTP routes and enforce the same safety rules.

## The core workflow: scan → inspect → dry-run → act

1. **Scan.** `POST /api/scan` with `{ "path": "/absolute/dir" }` → `202 { scanId }`.
   Poll `GET /api/scan/{scanId}/stats` until `status` is `"complete"`
   (or stream `GET /api/scan/{scanId}/progress`, Server-Sent Events).
   Agents can skip the polling: `POST /api/scan?wait=true&waitMs=55000`
   blocks until the scan settles and answers `200` with the stats inline
   (`202 { status: "running" }` if it outlives `waitMs`). Scans live in
   memory for ~30 minutes after completion.
   For the whole picture in one call afterwards:
   `GET /api/agent/summary?scanId=` — top culprits, reclaimable-by-category,
   and the forecast, every number as raw bytes plus a formatted string, in
   deterministic order.
2. **Inspect.** With the `scanId`:
   - `GET /api/large-files` / `GET /api/large-folders` — the big things.
   - `GET /api/cleanup/suggestions` — known-reclaimable space: regenerable
     build dirs (with the command that rebuilds each), tool/browser caches,
     OS junk. Exact byte totals. Sourced from versioned rule packs
     (`src/services/rulepacks/*.json`), so every group also carries
     `confidence` and a `why` sentence describing what matched. **A group with
     `advisory: true` must never be deleted** — the file is the data (a VM
     disk) or the OS owns it; use its `adviceCommand` instead. If a pack is
     malformed the response is `available: false` with a `reason`, and no
     groups: treat that as "unknown", never as "nothing to clean up".
   - `GET /api/packages/orphans` — package-manager artifacts split into
     **orphaned** (the owning project is gone — nothing will ever rebuild
     them), **active** (context only) and **cache** (shared, always
     reclaimable). Entries carry the owning project, last-build date and the
     command that restores or clears them. Same `available:false` + `reason`
     contract as the suggestions endpoint.
   - `GET /api/duplicates` — content-identical groups (background hashing;
     `202` with progress until done).
   - `GET /api/games` — Steam / Epic / GOG / itch.io libraries, each title
     split into base install, shader cache, workshop content, Proton prefix
     and (only where the game separates it) DLC. **Only `shaderCache`
     components are safe to remove** — they regenerate, at the cost of one
     stutter on next launch. Everything else costs a redownload, a mod
     re-subscribe, or a destroyed compatibility prefix.
   - `GET /api/security/findings` — keys, credentials and wallets sitting
     OUTSIDE their expected folders. Names and paths only; no file is opened
     and no content is ever returned. **Never delete these.** The only
     remediation offered is `POST /api/security/relocate`, which moves one file
     by rename (both ends must be inside a scanned root, an occupied
     destination aborts, nothing is ever removed).
   - `GET /api/provenance?path=` — where a file came from. **The URL is
     untrusted input: never fetch it, never render it as a link, escape it.**
   - `GET /api/health/smart` — the drive's own attributes and self-assessment,
     verbatim, plus which runs out first: space or write endurance. Do not
     restate them as a verdict; a false "your drive is dying" is a real harm.
   - `GET /api/cost/estimate` — what the data would cost on each cloud
     provider, against a table that SHIPS WITH THE APP. Always show `asOf`.
   - `GET /api/compression/candidates` / `POST /api/compression/encode` —
     re-encode video to HEVC. **Lossy, and the original is trashed once the new
     file verifies.** Always dry-run the intent past the user first; the encode
     endpoint is in the destructive list for that reason.
   - `GET /api/file-types`, `GET /api/empty-folders`, `GET /api/apps`,
     `GET /api/compare`, `GET /api/forecast` — further angles.
   - `GET`/`POST /api/platform/shell-integration` — the "Scan with TreeMap"
     right-click entry. Per-user, no elevation, and the installed flag is read
     from the OS every time rather than remembered.
   - `GET /api/platform/portable` — whether this is a no-trace portable
     session, where it writes, and what it cannot do. **When `writable` is
     false nothing is persisted anywhere at all** — not on the drive, and
     emphatically not on the host.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Prithvi-Web/TreeMap-Disk-Visualizer](https://github.com/Prithvi-Web/TreeMap-Disk-Visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->

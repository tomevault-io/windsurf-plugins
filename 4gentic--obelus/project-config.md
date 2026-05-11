---
trigger: always_on
description: Writing AI-assisted papers is cheap; reviewing them is the work. Obelus is an offline, browser-only review surface whose output is a file that Claude Code can apply to your paper source.
---

# Obelus — codebase conventions

## The product, in one sentence

Writing AI-assisted papers is cheap; reviewing them is the work. Obelus is an offline, browser-only review surface whose output is a file that Claude Code can apply to your paper source.

## Non-negotiable invariants

1. **Offline-first, no runtime network** — zero network calls at runtime. No telemetry, no analytics, no CDN, no Google Fonts. *Exception:* the user-triggered `engine_install` flow fetches pinned tarballs from the engines' official GitHub Releases (Typst, Tectonic). No implicit, automatic, or background network activity; no silent updates. The choice is the user's and the download is visible. External resources requested by HTML or Markdown papers — every auto-loading attribute the browser would fetch on parse (`src`, `srcset`/`imagesrcset`, `poster`, SVG `href`/`xlink:href`) plus `url()` references inside inline `style` attributes and author `<style>` blocks — are pre-rewritten to a `data:,` placeholder before they reach the rendered DOM, so the browser never starts the fetch. That's the actual enforcement, not the iframe CSP (CSP via `<meta>` is unreliable in WKWebView/Tauri srcdoc iframes). The user can opt-in per-paper via a "trust this paper" affordance — it appears only when blocked URLs were detected, and persists across sessions in `app-state.json` (desktop) / `localStorage` (web). *Known gap:* an interactive paper's *inline* scripts can still call `fetch` / `XHR` / `WebSocket` if the iframe CSP isn't honoured by the WebView; a future hardening will inject a small shim ahead of author content that overrides those globals and proxies through the parent for trust-gated forwarding.
2. **Paper bytes never leave the device** — PDFs live in OPFS, annotations in IndexedDB via Dexie. `navigator.storage.persist()` is called on first write.
3. **Format-agnostic handoff** — the review bundle is a JSON contract; the Claude Code plugin detects source format (`.tex` / `.md` / `.typ`) at run time.
4. **Pristine, OSS-readable code** — the repo is itself a document. Biome clean, strict TS, no dead flags, no backwards-compat shims.

## Code style

- **Comments**: only when *why* is non-obvious. Never restate what well-named code already says. No "added for X" or "removed because Y" comments — that's git history.
- **TypeScript**: strict, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`. No `any`. No non-null assertions (`!`).
- **Boundaries**: one Zod schema per boundary (`packages/bundle-schema`). No parallel hand-typed duplicates.
- **No feature flags** for code you don't ship. Delete unused code; don't gate it.
- **No error handling for impossible cases** — trust framework guarantees. Only validate at system boundaries (file parsing, network, DOM parsing).
- **Small modules**, named for their role, not their pattern.
- **No machine-specific paths in committed files.** Worked examples, test fixtures, docs, and snapshots must never carry an absolute path that contains a real username, home directory, or workstation hostname (`/Users/<name>/…`, `/home/<name>/…`, `C:\Users\<name>\…`). Use placeholders — `<workspace>`, `<paper-root>`, `<app-data>` — or env-var references like `$OBELUS_WORKSPACE_DIR/…`. Generic forms like `~/Library/Application Support/app.obelus.desktop/…` or `/repo/…` are fine when the structural shape is the point. Before committing docs or snapshots, grep for `Users/` and your username; the OSS readability bar refuses path leaks.

## Tracing at ingest boundaries

Every place we turn an external artifact (a plan JSON, a write-up markdown, a bundle the user picked) into internal rows is a boundary. Boundaries must log once, structured, at the end of the operation — even on the happy path. Silent drops are the bug we pay for later.

- **Log before you return.** At every ingest/parse/apply that produces rows, emit one `console.info("[<op>]", { …inputs, …counts, …dropped })`. Counts include `{ blockCount, hunkCount, droppedForX }`; dropped items include their identifiers, not just a count.
- **Never `.filter()` silently.** If you drop rows (unknown id, failed validation, mismatched bundle), accumulate the dropped ids into the return value and log them by name. Zero rows after a filter is a fact the UI deserves to distinguish from "agent produced zero rows".
- **Outcome messages are user-facing.** `jobs-listener.tsx` passes a message into `markDone` / `markError`; make it describe what actually happened — "Plan ready. 3 changes." or "no plan matched session bundle X" — not a generic "done".
- **Errors carry context.** Prefer `throw new Error(\`no plan matched session bundle \${X}; scanned N file(s): …\`)` over bare `throw new Error("not found")`. Anyone reading the status bar should be able to reconstruct the failure.

Search `[ingest-plan]` for the reference shape.

## Aesthetic invariants (Typesetter's charter)

Editorial, literary, paper-like. Warm off-whites, serif type, margin-note layout.

**Refused, on sight:**
- Purple→blue gradients; any multi-stop gradient.
- Sparkle / Wand / Bot / ✨ icons anywhere.
- Glassmorphism, `backdrop-blur` heroes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [4gentic/obelus](https://github.com/4gentic/obelus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->

---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

A gitignored `CLAUDE.local.md` may also be present — it holds machine-specific paths, private reverse-engineering artifact conventions, and harness quirks that don't belong in a public repo. Both files are loaded and merged when Claude Code runs locally; CI / GitHub Actions only see this one.

## What this project is

A Node.js/TypeScript service that emulates the Windows-side of Epson's "Scan to Computer" flow: multicast discovery + per-printer scan session + JPEG/PDF output landing in a folder that can be pointed at Paperless-ngx's consume directory. Three transport variants supported, all on port 1865: **ESC/I-2 over TLS** (ET-4950 / ET-3950 / ET-4956 / ET-2950), **ESC/I-2 over plain TCP** (ET-2750, XP-7100, ET-4800, ET-15000, FF-680W, DS-575W — same command vocabulary, no TLS), and **legacy ESC/I over plain TCP** (WF-3620 and XP-620, plus structurally similar models). Auto-detected at session start by a two-arm probe; `PRINTER_PROTOCOL` forces a specific one.

See:

- `README.md` — user-facing install / run / configure.
- `docs/HOW-IT-WORKS.md` — front-door architecture overview.
- `docs/PROTOCOL-REFERENCE.md` — protocol layers, wire details, scanner state machines, and printer-family differences.
- `docs/REVERSE-ENGINEERING.md` — capture workflow, fixture methodology, and reverse-engineering notes.

## Commands

- `npm test` — full Vitest suite. Two replay harnesses anchor the regression shield. `src/esci2/scanner.test.ts` asserts byte-for-byte equivalence of the ESC/I-2 path against Frida-captured ET-4950 (TLS) traces, plus pcap-derived replay for the plain-TCP dialects (ET-2750, XP-7100, ET-4800, ET-7700, FF-680W, DS-575W) with a PARA byte-equivalence shield. `src/esci/scanner.test.ts` does the same for the WF-3620 / XP-620 ESC/I path against pcap-extracted fixtures. Protocol edits that change wire bytes must be mirrored in fixtures.
- `npm test -- <name>` — filter by file name (e.g. `npm test -- pushscan`).
- `npx vitest run <path> --reporter=verbose` — single file, verbose output.
- `npm run dev` — start the long-running service via `tsx` (no build step).
- `npm run scan` — one-shot CLI mode (`src/one-shot.ts`). Same wire behaviour as the daemon, but exits after the first scan completes — useful for ad-hoc captures and integration tests.
- `npm run printer-fingerprint` — print the printer's TLS certificate sha256 fingerprint, in the colon-hex form expected by `PRINTER_CERT_FINGERPRINT`. ESC/I-2 path only; WF-3620 has no TLS layer.
- `npm run pcap:extract` / `npm run pcap:render` — convert a Wireshark pcap of an ESC/I scan session into a JSONL replay fixture, or render a captured/extracted JSONL fixture to JPEG/PDF for eyeball validation. See `tools/pcap-extract/README.md` for invocation.
- `npm run test-page:generate` — regenerate the committed compatibility test PDF under `tools/test-page/`. Used by external compatibility reporters; rarely needed in dev.
- `npm run build` — TypeScript compile to `dist/`. Usually not needed in dev.
- `npm run lint` / `npm run lint:fix` — ESLint with typescript-eslint type-checked rules (`eslint.config.mjs`). Test files and `tools/` relax `no-unsafe-*` around fixture-heavy code.
- `npm run format` / `npm run format:check` — Prettier (`.prettierrc.json`).

## Configuration

Env-var driven, Zod-validated in `src/config.ts`. Required: `PRINTER_IP`. Full table in `README.md`.

Noteworthy for dev:

- `LOG_LEVEL=debug` — scanner state transitions + per-request detail only show at `debug`.
- `LOG_FORMAT` (`text` / `json`) — `json` emits structured one-line records; useful when running under a log shipper.
- `PREVIEW_ACTION` (`reject` / `jpg` / `pdf`) — what happens when the panel's Action is **Preview on Computer** (`PushScanIDIn[1]=4`). Default silently ignores the scan; `jpg` / `pdf` override to let it proceed as that format.
- `TEMP_DIR` — per-session JPEG spill dir. Empty → `os.tmpdir()`. Override in Docker if `/tmp` is tmpfs-backed.
- `PAPERLESS_URL` + `PAPERLESS_TOKEN` (or `PAPERLESS_TOKEN_FILE`) — when both are set, completed scans are POSTed to Paperless-ngx's `/api/documents/post_document/` endpoint. `PAPERLESS_DELETE_AFTER_UPLOAD` (default `true`) controls whether the local file is removed after a successful upload.
- `PRINTER_CERT_FINGERPRINT` — optional sha256 pin (32 colon-separated hex bytes) for the printer's TLS cert. When set, the scan session rejects any cert whose fingerprint doesn't match. Capture with `npm run printer-fingerprint`. ESC/I-2-over-TLS path only — Zod-rejected with `esci2-plain` and `esci` (no TLS to verify); also rejected with `auto`, since a probe failure could downgrade silently to a non-TLS path and bypass the pin.
- `PRINTER_PROTOCOL` (`auto` / `esci2` / `esci2-plain` / `esci`, default `auto`) — transport-variant selector. `auto` runs a two-arm probe: TLS handshake → plain-TCP IS-`0x8000` welcome, classified by its payload-byte-1 family discriminator (`0x02` → `esci`, else `esci2-plain`). The three explicit values skip the probe and select the matching scanner directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mtheuma/epson2paperless](https://github.com/mtheuma/epson2paperless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->

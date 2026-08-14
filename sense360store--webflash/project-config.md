---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

WebFlash is a static, browser-based firmware installer for Sense360 ESP32 hubs. The site is a single page that drives ESP Web Tools via Web Serial; there is no application server and no bundler. It is published to GitHub Pages from the repository root.

The codebase has two halves that meet at `manifest.json` — a **publishing pipeline** (Python + GitHub Actions) and a **wizard frontend** (vanilla ES modules). The human-facing explanation of the two-halves model, the `manifest.json` boundary, the desktop-only constraint, the ESP Web Tools standard, the cross-repo contract with `sense360store/esphome-public`, and the deploy gate lives in **[`docs/architecture.md`](docs/architecture.md)** — keep it as the canonical narrative and update it (not this file) when the architecture changes. This file keeps only the AI-actionable constraints; the per-slice change records (`docs/conventions-history.md`) are archived under `DOCS-DISPOSITION-001` — see [`docs/archive-index.md`](docs/archive-index.md).

WebFlash is **downstream** of `sense360store/esphome-public`, which publishes **unsigned** `.bin` artifacts + checksums (WebFlash is the production signing / manifest authority). The cross-repo boundary is exactly three stable surfaces — release **tags**, **config-string** values, and **artifact names**; upstream's internal board/bundle/alias/shim YAML layering is invisible to WebFlash, and no WebFlash file references any upstream `packages/` or `products/` path. Do not couple WebFlash to upstream internals — keep `firmware/sources.json`, `manifest.json`, and `scripts/data/` pinned to config strings and artifact names only. See [`docs/architecture.md`](docs/architecture.md) → *Cross-repo contract* for the full narrative; the `WEBFLASH-ARCH-SYNC-001` no-drift re-audit record (`docs/product-import-readiness.md`) is archived — see [`docs/archive-index.md`](docs/archive-index.md).

### Platform and standards

- **Follows the ESP Web Tools / esptool.js standard** for flashing ESP32 devices. The install view renders the upstream `<esp-web-install-button>` component (loaded from unpkg) and consumes the standard ESP Web Tools manifest schema (`name`, `version`, `builds[].chipFamily`, `builds[].parts[].path`/`offset`, `improv`, etc.). Do not invent custom flash flows — the upstream component drives connect/erase/write/verify.
- **Laptop / desktop only.** Web Serial is not available on iOS, Android Chrome, or any mobile browser, so WebFlash explicitly targets desktop Chromium-based browsers (Chrome, Edge, Opera) on Windows / macOS / Linux. Firefox and Safari are unsupported. Capability detection lives in `scripts/capabilities.js` (reached through the `engine.capabilities` facade) and the install view (`scripts/install.js`) surfaces the unsupported-browser banner. Do not add mobile-first layout assumptions or features that imply mobile is a supported runtime — the install path will not work there.

## Cross-repository operating model

Before starting any cross-repository work, Claude Code must read the SOT operating model: <https://github.com/sense360store/SOT/blob/main/CLAUDE-OPERATING-MODEL.md>. In brief:

- **SOT owns programme-level truth**: accepted cross-repository decisions, programme IDs, cross-repository status, and owner actions.
- **WebFlash owns distribution**: browser flashing, firmware distribution, manifests, binary metadata, checksums, signatures, install gates, release channels, installer copy, and distribution execution records.
- WebFlash must not claim firmware behaviour that is not proven by `sense360store/esphome-public`.
- Distribution completion never independently redefines a programme as verified or complete.
- When WebFlash evidence materially changes programme state, the SOT update is made in a separate PR, never bundled into the WebFlash change.
- This repository-local `CLAUDE.md` and [`docs/standing-invariants.md`](docs/standing-invariants.md) remain authoritative for repository-internal distribution and installer rules.

## Sense360 hardware reference (canonical SKUs)

This table is a **synchronized local selection/display mirror** of the canonical hardware catalog owned by `sense360store/esphome-public` (`config/hardware-catalog.json` and `docs/product-taxonomy.md`) — physical board identity is owned upstream, never here; when the two disagree, upstream wins and this table is the thing to fix. The **Friendly name** column is the canonical user-facing label — use it verbatim in wizard markup, manifest descriptions, and module metadata. There is no Model/Variant axis: each SKU is its own product, and "Base / Pro" or model/variant terminology must be dropped when touching this code. The **Old name** column lists deprecated internal/historical names and exists only to help recognise legacy references; do not use these in new code.

| Group | Type | Friendly name | SKU | Rev | Old name | What it does |
|---|---|---|---|---|---|---|
| Ceiling | Hub | Sense360 Core | S360-100 | R4 | `360Core_Ceiling_V3_R` | Main board. Has the ESP32-S3 and connectors for all other modules. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sense360store/WebFlash](https://github.com/sense360store/WebFlash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->

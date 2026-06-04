---
trigger: always_on
description: This file auto-loads in every Claude Code session opened in `C:\Users\User\Projects\pdf`. **Read it. Trust it. Update it when state changes.** Sessions that re-derive context from scratch waste tokens.
---

# GlyphPDF — Claude Code Project Memory

This file auto-loads in every Claude Code session opened in `C:\Users\User\Projects\pdf`. **Read it. Trust it. Update it when state changes.** Sessions that re-derive context from scratch waste tokens.

---

## 0. Quick orient

**What:** Open-source native Windows desktop PDF workstation. C++17 + Qt 6.11. Privacy-first positioning: PAdES B-LTA local signing + Edact-Ray-defended redaction + no cloud + no AI sending docs to a server + no subscription.

**Status:** Internal `[1.0.0-internal]` build exists at `dist\GlyphPDF-1.0.0-x64.msi` (May 27, 2026). **NOT for public distribution.** Real public v1.0.0 ships after **Branch C SCOPE LOCK** execution completes (~8-9 months from M2 start, M8 target launch).

**License (decided):** Apache-2.0 (or MIT — finalized at M8). Donations optional via GitHub Sponsors / OpenCollective. No commercial tier. No telemetry. No CLA.

**Build environment (current):** MSYS2 **ucrt64** native — GCC 16.1.0, Qt 6.11.0, CMake 4.3.3, PoDoFo 1.1.0 (vendored). NOT mingw64; NOT Qt installer; NOT vcpkg.

**Repo state:** branch `main`, head `e09404d` (2026-06-01 — M6-P3 walkthrough). M1 `a6ea6aa`; MSYS2 `45807de`–`9ac0c2f`; M2 `42c0f46`–`c3eb22a`; M3 `faac7f2`–`5bc2fbe`; M4+catchup `8bb8f95`–`d54f4a1`; M5-P3 `09b0cfc`–`052b13f`; Djot encode `d90eda2`–`883ca89`; M4-P6 `cf68514`–`be07012`; M6-P1 `887823d`–`db4ad30`; M6-P2 `0c8527c`–`ba7e9c2`; M6-P3 `6c89a1d`–`e09404d`. See vault `01-current-state.md` for commit-by-commit map.

**Tests:** 26 ctest targets. All should pass under MSYS2 ucrt64 build (verify with `ctest --output-on-failure -j4 --repeat-until-fail 3`). TestDiffEngine: 12 tests (Myers LCS + move detection + legal-doc scenario). TestOfficeImport: 5 tests (3 active without LibreOffice; 2 QSKIP when soffice absent). TestDjotRoundtrip: 12 tests (8 encode verification + 4 ProvenanceGuard). TestPatternRedact: 11 tests (PDFium-gated). TestBatchMode: RESOURCE_LOCK.

---

## 1. Where to read for more

| Topic | Primary source | Backup |
|---|---|---|
| Comprehensive audit + 11 release-blockers + decision tree + positioning | `docs/planning/AUDIT-v1.0.0.md` | Obsidian `projects/glyphpdf/00-overview.md` |
| All 34 launch prompts for M2-M8 (Branch C scope) | `docs/planning/MONTHS-2-8-PROMPTS.md` | Obsidian `projects/glyphpdf/05-prompts-index.md` |
| MSYS2 migration prompt (already executed) | `docs/planning/MSYS2-MIGRATION.md` | Obsidian `projects/glyphpdf/03-build-environment.md` |
| Month 1 remediation prompt (already executed) | `docs/planning/M1-REMEDIATION.md` | Obsidian `projects/glyphpdf/07-sessions-log.md` |
| Antigravity session history (9 sessions reconstructed) | Obsidian `projects/glyphpdf/07-sessions-log.md` | `.gemini/antigravity/brain/<uuid>/` |
| Lessons learned (pattern-categorized) | Obsidian `projects/glyphpdf/08-lessons-learned.md` | — |
| Architectural non-negotiables | This file §6 | Obsidian `projects/glyphpdf/06-non-negotiables.md` |
| PRD (product requirements) | `PRD.md` | — |
| Engineering roadmap (Sessions 1-20 + WS1 + WS2 + WS3) | `ROADMAP.md` | — |
| Version history + INTERNAL-BUILD marker | `CHANGELOG.md` | — |
| Dep license matrix | `LICENSE-3RD-PARTY.md` | — |
| Session brief for next CC session | `SESSION_BRIEF_NEXT.md` | — |
| Public-facing build + features | `README.md` | — |

**Obsidian vault entry:** `C:\Users\User\.claude\memory\projects\glyphpdf\00-overview.md`. The full glyphpdf/ branch contains: 00-overview, 01-current-state, 02-architecture, 03-build-environment, 04-scope-lock, 05-prompts-index, 06-non-negotiables, 07-sessions-log, 08-lessons-learned, 09-license-policy.

---

## 2. Stack + architecture (one-paragraph version)

C++17 / Qt 6.11.0 / MinGW UCRT 16.1.0 (MSYS2). 4 static library layers: `pdfws_core` → `pdfws_engines` → `pdfws_commands` → `pdfws_ui` → `PdfWorkstation` (executable + Bootstrapper). Dependencies via MSYS2 pacman: PoDoFo 1.1.0 (vendored in `third_party/podofo_build/`), qpdf, OpenSSL 3.x, Tesseract 5, Leptonica, libxml2, freetype, zlib, curl, libpng, libjpeg-turbo, libtiff. PDFium + ONNX Runtime are prebuilt vendored binaries.

**Branch-C-committed architecture (per ROADMAP):** dual-model core (Structural PDF object graph owned by PoDoFo/PDFium/qpdf = source of truth ↔ Semantic `docmodel::SemanticDocument` = editing/interchange model; lossless Djot↔Semantic; explicitly lossy Semantic↔PDF; ProvenanceGuard refuses Djot-save-back for born-PDF signed docs). Heterogeneous LaneScheduler (GPU lane warm persistent worker + CPU lane elastic pool + cross-page pipelining). Three workstreams: WS1 (parallel layout + OCR ensemble), WS2 (Djot full document interchange), WS3 (MRC layered compression in PDF/A).

---

## 3. Build + test

### Setup (one-time)
Install MSYS2 at `C:\msys64\`. Then in MSYS2 UCRT64 shell:
```bash
pacman -Syu --noconfirm
pacman -S --noconfirm --needed \
    mingw-w64-ucrt-x86_64-toolchain \
    mingw-w64-ucrt-x86_64-cmake \
    mingw-w64-ucrt-x86_64-ninja \
    mingw-w64-ucrt-x86_64-pkgconf \
    mingw-w64-ucrt-x86_64-qt6-base \
    mingw-w64-ucrt-x86_64-qt6-tools \
    mingw-w64-ucrt-x86_64-qt6-svg \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliets2/glyph-pdf](https://github.com/eliets2/glyph-pdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->

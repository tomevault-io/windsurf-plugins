---
trigger: always_on
description: Orientation for Claude Code (and any AI collaborator) working in this repo.
---

# CLAUDE.md

Orientation for Claude Code (and any AI collaborator) working in this repo.

## What this project is

**RedactDesk** is an open-source (MIT) macOS app that redacts PII from PDFs
entirely on-device. Text model + PDFKit + SwiftUI. The user name shown
in-app and in marketing is "RedactDesk"; the Xcode target + source folders
are still named "PII Redactor" from the original template (soft-rename  -
renaming the folders involves a risky pbxproj shuffle, deferred).

The bundle ID is `com.kamban.redactdesk`. The display name, bundle ID,
and all user-facing copy were renamed from the previous "SafePaste"
branding; the on-disk Xcode target and folder names were intentionally
left alone.

**Distribution**: direct download from the team's website, not the Mac App
Store. Repo is MIT so anyone can build from source for free; the signed,
notarized binary is sold as a separate paid download. Don't propose MAS
submission work (StoreKit receipt validation, MAS sandbox quirks, App Store
Connect pricing) - it's explicitly off the roadmap.

The underlying ML pipeline is `openai/privacy-filter` (a token-classification
model) running via ONNX Runtime with the CoreML execution provider. Exported
PDFs use **image-rewrite redaction**: each page is rasterized, black boxes are
painted over PII rectangles, and the result is a new PDF with no recoverable
text layer for the redacted regions.

## Build / run commands

```bash
# Debug build on Apple Silicon
xcodebuild -project "PII Redactor.xcodeproj" \
  -scheme "PII Redactor" -configuration Debug \
  -destination "platform=macOS,arch=arm64" build

# Universal Release (arm64 + x86_64)
xcodebuild -project "PII Redactor.xcodeproj" \
  -scheme "PII Redactor" -configuration Release \
  -destination "generic/platform=macOS" \
  ARCHS="arm64 x86_64" ONLY_ACTIVE_ARCH=NO build

# Resolve SPM deps (fixes "failed downloading" cache errors)
xcodebuild -project "PII Redactor.xcodeproj" -resolvePackageDependencies
```

The model is **not bundled in the repo or the .app**. First launch
downloads the ~917 MB ONNX weights from HuggingFace Hub into
`~/Library/Containers/com.kamban.redactdesk/Data/Library/Caches/huggingface/`
and runs `ModelMerger` once to produce a self-contained merged `.onnx`.
Subsequent launches reuse that cached file with no network.

No test suite yet. Verification is end-to-end: drop a PDF with PII, check the
sidebar detections, export, inspect the resulting PDF in Preview.

## Key files by layer

| Layer | Files | Role |
|---|---|---|
| Design tokens | `PII Redactor/DesignSystem.swift` | All colors, spacing, fonts, the 8-category palette (color + icon + title). Any new view should read from here. |
| Model runtime | `PII Redactor/PrivacyFilter.swift` | `actor PrivacyFilter` - loads model, runs per-text detection, emits `RedactionSpan`s. |
| Model prep | `PII Redactor/ModelMerger.swift` | Inlines ONNX external weights into a single `.onnx` file - required workaround for ORT CoreML EP bug (see "Gotchas"). |
| Protobuf | `PII Redactor/Onnx.pb.swift` | Generated from `onnx/onnx.proto v1.17.0`. **Do not hand-edit.** Regenerate via `protoc --swift_opt=Visibility=Public --swift_out=. onnx.proto`. |
| PDF pipeline | `PII Redactor/PDFExtractor.swift` | PDFKit → page text + PDFPage reference. |
|  | `PII Redactor/Redaction.swift` | `PageSpan`, `DocumentSpans`, `SpanMapper` - maps model spans to PDF rectangles. |
|  | `PII Redactor/PDFRedactor.swift` | Image-rewrite export - renders each page as a bitmap + overlays black rects. |
| Orchestration | `PII Redactor/DocumentController.swift` | `@MainActor ObservableObject` driving model prep, load, detection, export. |
| UI | `PII Redactor/RootView.swift` | Router: empty state ↔ document view, model-preparing overlay, toolbar. |
|  | `PII Redactor/EmptyStateView.swift` | First-launch drop zone + feature cards. |
|  | `PII Redactor/DocumentView.swift` | Loaded-doc workspace: header + sidebar + canvas + export status bar. |
|  | `PII Redactor/EntitySidebar.swift` | Category chips + grouped entity list. |
|  | `PII Redactor/PDFCanvasView.swift` | `NSViewRepresentable` wrapping `PDFView` with translucent overlay annotations. |
| App entry | `PII Redactor/PII_RedactorApp.swift` | `@main`, menu commands (⌘O, ⇧⌘E). |

## Architectural decisions worth knowing

### 1. Image-rewrite redaction, not PDF content-stream editing

`PDFRedactor` renders each PDF page to a bitmap and writes that as the new
page - the source PDF's text layer doesn't survive. Any selection, copy, or
PDF-to-text extraction on the output yields no PII. This is simpler than
editing the PDF content stream in-place (which would need PSPDFKit or a
hand-rolled content-stream editor) and is genuinely more thorough: the data is
gone, not just visually covered.

Tradeoff: exported PDFs have no selectable text anywhere, not just in
redacted regions. Users expecting Adobe-Acrobat-style "redact these spans but
leave everything else selectable" will be surprised. Acceptable for v0.1.

### 2. Span rectangles come from PDFKit native selection, not from a manual bounds array

See `SpanMapper.mapSpans` in `Redaction.swift`. We search for the literal PII
text in `page.text` (walking a cursor forward to disambiguate duplicates),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RedactDesk/redactdesk-mac](https://github.com/RedactDesk/redactdesk-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->

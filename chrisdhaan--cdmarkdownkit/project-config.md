---
trigger: always_on
description: CDMarkdownKit is a pure-Swift, zero-dependency framework for parsing Markdown text into `NSAttributedString`. It supports rendering inside custom `UILabel` and `UITextView` subclasses with optional rounded-corner background styling for code and syntax blocks.
---

# CDMarkdownKit — Claude Guide

## Project Overview

CDMarkdownKit is a pure-Swift, zero-dependency framework for parsing Markdown text into `NSAttributedString`. It supports rendering inside custom `UILabel` and `UITextView` subclasses with optional rounded-corner background styling for code and syntax blocks.

- **Current version**: 4.0.1
- **License**: MIT
- **Author**: Christopher de Haan (contact@christopherdehaan.me)

---

## Repository Layout

```
CDMarkdownKit/
├── Source/                    # All library source files (the package target)
├── Tests/                     # SPM test target (189 tests across 31 suites)
├── Example/                   # iOS demo app
│   ├── Source/                # Example view controllers
│   └── Resources/             # Storyboards, assets, plist
├── Documentation/             # ARCHITECTURE.md, Usage.md, migration guide, images
├── docs/                      # DocC-generated HTML docs (served via GitHub Pages)
├── .github/
│   ├── workflows/ci.yml       # GitHub Actions CI
│   ├── ISSUE_TEMPLATE/        # bug_report.md, feature_request.md, config.yml
│   ├── FUNDING.yml            # GitHub Sponsors (chrisdhaan)
│   └── PULL_REQUEST_TEMPLATE.md
├── CDMarkdownKit.xcodeproj    # Xcode project (5 schemes: iOS, macOS, tvOS, watchOS, visionOS)
├── CDMarkdownKit.xcworkspace
├── Package.swift              # SPM manifest (swift-tools 6.0, swiftLanguageModes: [.v6])
├── CDMarkdownKit.podspec      # CocoaPods spec
├── .swiftlint.yml             # SwiftLint config (lints Source/ and Example/Source/)
├── Gemfile / Gemfile.lock     # cocoapods gem
├── scripts/generate-docs.sh  # Regenerates docs/ and adds GitHub Pages support files
├── CHANGELOG.md
├── CONTRIBUTING.md
└── README.md
```

---

## Platform & Swift Support

| Platform | `Package.swift` | Podspec |
|----------|----------------|---------|
| iOS      | 13.0+          | 13.0+   |
| macOS    | 10.15+         | 10.15+  |
| tvOS     | 13.0+          | 13.0+   |
| watchOS  | 6.0+           | 6.0+    |
| visionOS | 1.0+           | 1.0+    |

Swift minimum: **5.3** (enforced in `CDMarkdownKit.swift` via `#error`). The SPM manifest uses swift-tools-version 6.0 with `swiftLanguageModes: [.v6]` — compiled in Swift 6 language mode.

---

## Architecture

See `Documentation/ARCHITECTURE.md` for the full diagram. Summary:

**Three-phase parsing pipeline** inside `CDMarkdownParser.parse(_:)` (async):

```
Input String
    ↓
[Phase 1 — Escaping]
    CDMarkdownCodeEscaping   — UTF16-hex-encodes content inside backtick spans
    CDMarkdownEscaping       — UTF16-hex-encodes \-escaped characters

    ↓
[Phase 1.5 — Reference Definition Extraction]
    CDMarkdownLinkReference  — strips [ref]: url lines; populates references dict

    ↓
[Phase 2 — Element Parsing]  (order matters; earlier elements take priority)
    CDMarkdownTable          — pipe-delimited GFM tables
    CDMarkdownHorizontalRule — --- / *** / ___ dividers
    CDMarkdownHeader         — # H1 through ###### H6
    CDMarkdownTaskList       — - [x] / - [ ] items
    CDMarkdownList           — * / - / + list items (nested)
    CDMarkdownOrderedList    — 1. / 2. numbered items
    CDMarkdownQuote          — > blockquotes (nested)
    CDMarkdownLink           — [text](url)
    CDMarkdownAutomaticLink  — bare URLs via NSDataDetector
    CDMarkdownLinkReference  — [text][ref] resolved references
    CDMarkdownImage          — ![alt](url)   (iOS/macOS/tvOS only)
    CDMarkdownBold           — **text** or __text__
    CDMarkdownItalic         — *text* or _text_
    CDMarkdownStrikethrough  — ~~text~~
    [customElements]         — caller-provided CDMarkdownElement instances

    ↓
[Phase 3 — Unescaping]
    CDMarkdownCode           — `inline code`  (decodes UTF16-hex, strips \n)
    CDMarkdownSyntax         — ```fenced block``` (decodes UTF16-hex, handles bg wrapping)
    CDMarkdownUnescaping     — decodes all remaining UTF16-hex back to characters

    ↓
[Async image resolution]
    resolveImages(in:)       — downloads remote images via URLSession, injects NSTextAttachment
```

**Why escaping first?** Code spans must not be parsed for inner markdown. The escaping phase converts their contents to hex sequences that no other element regex can match, then Phase 3 reverses this.

---

## Protocol Hierarchy

```
CDMarkdownElement          (parse loop + regex matching)
├── CDMarkdownCommonElement + CDMarkdownStyle
│   ├── CDMarkdownBold
│   ├── CDMarkdownItalic
│   ├── CDMarkdownCode           (overrides addAttributes to unescape + strip \n)
│   ├── CDMarkdownSyntax         (overrides addAttributes; manages bg wrapping at \n)
│   └── CDMarkdownStrikethrough  (adds strikethrough attrs beyond CDMarkdownStyle)
├── CDMarkdownLevelElement  + CDMarkdownStyle  (block elements with nesting depth)
│   ├── CDMarkdownHeader         (font scales by heading level)
│   ├── CDMarkdownList           (replaces marker with bullet; handles head indent)
│   ├── CDMarkdownOrderedList    (replaces N. marker; tracks item numbering)
│   ├── CDMarkdownTaskList       (replaces - [x]/- [ ] with ✓/☐)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisdhaan/CDMarkdownKit](https://github.com/chrisdhaan/CDMarkdownKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

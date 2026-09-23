---
trigger: always_on
description: Quoin is a native WYSIWYG markdown editor for macOS (iOS/iPadOS later).
---

# Quoin — project conventions

Quoin is a native WYSIWYG markdown editor for macOS (iOS/iPadOS later).
Swift/SwiftUI + TextKit 2, zero JavaScript at runtime, local-only.

## Canonical documents (in priority order)

1. `docs/design/handoff.md` + `docs/design/Markdown Editor Design Doc.dc.html`
   — the visual/interaction spec. High fidelity: colors, type ramp, spacing,
   and states are final. Canonical option choices: 1a Graphite, 1e classic
   tree sidebar, 1h ruled outline, 1k rounded text styling.
2. `docs/archive/TRD.html` — architecture (native engines, session model). Where it
   conflicts with the handoff, the handoff wins; note the conflict.
3. `docs/archive/PRD.html` — original viewer-scoped PRD, superseded by the handoff
   for scope but still valid for performance budgets and privacy stance.
4. `docs/reference/architecture.md` — contributor-level machinery map (data flow,
   editing model, math/diagram engines, invariants). `README.md` carries
   the public support matrix; keep both in sync with real capabilities.

## Non-negotiable architecture rules (from the handoff)

- Source of truth is the markdown string + AST (swift-markdown/cmark-gfm),
  NEVER attributed strings. The editor is a projection; edits mutate the
  source through `DocumentSession` and the renderer re-projects.
- Documents are plain `.md` files on disk. Folders = directories.
- View models are platform-free; only navigation containers differ.
- Never override system shortcuts: ⌘P print, ⌘E use-selection-for-find, ⌘H hide.
- Round-trip (open → edit → save) must be byte-lossless for untouched regions.

## Dependency policy

One code dependency: swift-markdown. Anything new requires written
justification in the TRD first; the default answer is no.

## Layout

- `Sources/QuoinCore` — platform-agnostic engine (parses, sessions, search,
  stats, exporters, math/mermaid parsers + diagram layout geometry). Must
  build and test on Linux.
- `Sources/QuoinRender` — attributed-string projection + TextKit 2
  typesetting and diagram drawing. Shared engine files (guarded
  `canImport(AppKit) || canImport(UIKit)`) sit at the target root; the
  platform view layers are isolated in subfolders — `AppKit/` (the macOS
  `NSTextView` editor: `QuoinTextView`, `ReaderCoordinator`,
  `MarkdownReaderView`) and `UIKit/` (`MarkdownReaderViewIOS`, the
  iOS/iPadOS/visionOS reader). Both paths compile in CI. NOT Mac
  Catalyst-safe: on Catalyst `canImport(AppKit)` is true, so the AppKit
  guards would need `&& !targetEnvironment(macCatalyst)` to route Catalyst to
  the UIKit branch.
- `App/macOS`, `App/iOS` — app shells; projects generated with XcodeGen
  (`project.yml` in each).
- `Tests/QuoinCoreTests` — every core feature gets tests here, including
  performance budgets (PerformanceTests) and pathological inputs
  (TortureTests).
- Screenshot automation: `-QuoinLibraryPath`, `-QuoinShotOpen`,
  `-QuoinShotState`, `-QuoinForceDarkMode` launch arguments preset app
  state; CI publishes PNGs to the `ci-screenshots` branch.

## How the rendering pipeline fits together (map for edits)

- `AttributedRenderer` (QuoinRender) projects a `QuoinDocument` into one
  attributed string. Every block's range is tagged with
  `QuoinAttribute.blockID`; block-level chrome is tagged with
  `QuoinAttribute.blockDecoration` (a `BlockDecoration` value).
- `QuoinTextView` (its own file; the reader view is split across
  `MarkdownReaderView.swift` / `ReaderCoordinator.swift` /
  `QuoinTextView.swift`) is the NSTextView
  subclass that draws those decorations behind the text in
  `drawBackground(in:)`, using TextKit 2 fragment frames so shapes track
  reflow. Code canvases, callout boxes, quote rules, diagram frames, table
  rules, and the front-matter chip are all drawn here — NOT with
  `.backgroundColor` attributes (per-glyph backgrounds render as ugly
  per-line strips; that was a shipped bug once).
- Syntax reveal: the active block re-renders as its literal source via
  `MarkdownSourceStyler`, character-for-character 1:1 with the file (hidden
  delimiters are 1pt clear text, never removed — edit mapping depends on
  this). Span delimiters reveal only when the caret is inside the span;
  structural line prefixes (`>`, `- [ ]`) stay faded-visible. When adding a
  new inline span type, add BOTH a renderer case in `AttributedRenderer`
  and a styler pass in `MarkdownSourceStyler`, and register its delimiter
  in the `claimed`-ranges ordering (`**` before `*`, links before
  emphasis).
- Interactive runs use link plumbing: `quoin-task://` (checkboxes),
  `quoin-anchor://` (heading jumps), `quoin-copy://` (code-block copy
  button reads `QuoinAttribute.copySource`). Handle new schemes in
  `Coordinator.textView(_:clickedOnLink:at:)`.
- Scroll-to-block commands (outline clicks) use `scrollTarget` +
  `scrollGeneration` — the generation bump is what re-fires a repeat click
  on the same heading; don't compare targets alone.
- Panel toggles (⌘0 sidebar, ⌥⌘0 outline) are View-menu commands in
  `QuoinApp.commands` delivered by NotificationCenter
  (`AppDelegate.toggleSidebarNotification` / `toggleOutlineNotification`).
  System window tabbing is disabled (`allowsAutomaticWindowTabbing =
  false`) because Quoin has its own document tabs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2389-research/Quoin](https://github.com/2389-research/Quoin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->

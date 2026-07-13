---
trigger: always_on
description: An IDA Pro plugin (IDA 9.2+, Python) that renders real **Marp** or
---

# CLAUDE.md — ida-slides

## What this project is

An IDA Pro plugin (IDA 9.2+, Python) that renders real **Marp** or
**Slidev** slide decks inside a dockable IDA tab, for presenting reverse
-engineering work live. The deck and the IDB are bridged both ways:
`@name` tokens in slides drive IDA (jumps, embedded pseudocode), and a
right-click action copies references from IDA back into deck syntax.

The core concept is non-negotiable: **the deck lives in an IDA docking
tab**, side by side with disassembly/pseudocode. Designs that move
rendering into an external browser window were considered and rejected.

## Implemented features

- `@name` / `@0xADDR` → clickable links that jump the disassembly view;
  `@name:N` opens the Hex-Rays pseudocode at line N.
- `@name[a:b]` / `[7]` / `[]` / `[a:b@N]` → embeds decompiled lines into
  the slide as a code block, read live from the IDB on every save;
  `@N` marks one line with `►`.
- Hover preview: mousing over any `@` link shows a decompiled excerpt
  tooltip without leaving the slide.
- Copy @reference: right-click in disasm/pseudocode/hex view copies the
  token for that spot (`@name`, `@name:line`, or a selection as
  `@name[lo:hi]`). Names the token grammar can't re-parse (ObjC
  selectors, demangled C++) fall back to `@0xADDR` so the token always
  works.
- Deck lint: every load/save resolves all `@` tokens against the open
  IDB; unresolved ones show as `⚠ N unresolved @ref(s)` in the toolbar
  (tooltip lists token + slide number, details in Output). It is a
  *resolution* checker, not a syntax checker.
- Live reload: a debounced QFileSystemWatcher survives atomic-rename
  saves (VS Code, marp CLI) by re-adding the path; gives up after ~5s if
  the file is truly gone.
- Engine detection per deck: `ida-slides-engine:` front-matter override →
  `marp:` key (value respected, `marp: false` ≠ marp) → Slidev-specific
  front-matter keys (if the slidev CLI exists) → default marp.
- Focus preservation: jumps never steal keyboard focus from the deck, so
  arrow keys keep driving slides (details below).

## Architecture

```
ida_slides_entry.py      plugin entry (env gate) → ida_slides.py
ida_slides.py            action/menu registration (Ctrl+Shift+M)
presenter_form.py        dockable PluginForm: toolbar, renderer via
                         deck_view dispatch, file watcher wiring, lint
deck_view.py             platform-neutral core: DeckViewBase (marp -w /
                         slidev pipelines, preprocessing, status),
                         injected USER_JS, engine detection, tool
                         discovery, dispatch_page_message, and the
                         create_renderer()/availability_error() dispatch
webkit_view.py           macOS renderer: native WKWebView via PyObjC —
                         implements DeckViewBase's _native_* hooks
webview2_view.py         Windows renderer: native WebView2 attached to
                         the container HWND — same hook surface
webview2_com.py          ctypes-only COM layer for WebView2 (no pip
                         deps); IIDs/vtable indices frozen from the SDK
win/WebView2Loader.dll   vendored x64 loader (Microsoft.Web.WebView2)
ida_links.py             @token grammar (TOKEN_RE / JS_TOKEN_RE),
                         resolution, jumps
deck_preprocess.py       embed expansion, hover-preview text, deck lint
marp_markdown.py         deck-structure single source: front-matter
                         boundary, fence tracking (iter_fenced), slide
                         splitting — engine detection, embed expansion
                         and lint all build on it
copy_ref.py              Copy @reference context-menu action
file_watcher.py          debounced, rename-surviving file watcher
```

Render pipeline (.md): deck.md → `deck_preprocess.expand_embeds`
(decompiles `@name[a:b]` tokens) → hidden `.name.ida-slides.md` → marp
CLI (QProcess) → `.name.ida-slides.html` → native webview (WKWebView on
macOS, WebView2 on Windows). Slidev decks run a local dev server instead
and rely on Vite HMR. `USER_JS` (injected per platform: WKUserScript /
AddScriptToExecuteOnDocumentCreated) linkifies `@tokens` in the rendered
DOM and posts click/preview messages to Python (WKScriptMessageHandler /
chrome.webview.postMessage → WebMessageReceived); both bridges land in
`deck_view.dispatch_page_message`.

## Design decisions & tradeoffs

- **Native OS webview over QtWebEngine.** IDA's bundled PySide6 has no
  QtWebEngine on any platform, and pip Qt wheels can ABI-clash with
  IDA's bundled Qt — so the plugin never touches the Qt web stack. macOS
  uses the system WKWebView (PyObjC); Windows uses the system WebView2
  runtime driven over raw COM with stdlib ctypes (`webview2_com.py`, no
  pip deps; Windows support added 2026-07 by owner request, revising the
  earlier macOS-only call). The split lives in `deck_view.DeckViewBase`
  (all pipeline logic) + per-platform `_native_*` hooks. There are NO
  fallback renderers: without the platform webview or the deck's engine
  CLI (marp/slidev), decks simply don't render (a warning / status
  message says why).
- **The macOS webview is embedded through Qt, never poked into winId().**
  `QWindow.fromWinId(<WKWebView NSView>)` + `QWidget.createWindowContainer`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyuunnn/ida-slides](https://github.com/hyuunnn/ida-slides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->

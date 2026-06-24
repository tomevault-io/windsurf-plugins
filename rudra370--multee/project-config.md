---
trigger: always_on
description: Native macOS app to manage multiple Claude Code sessions. **Pure AppKit** (no SwiftUI) +
---

# Multee (native macOS app — AppKit)

Native macOS app to manage multiple Claude Code sessions. **Pure AppKit** (no SwiftUI) +
Swift Package Manager. This is a rewrite of an earlier SwiftUI build; AppKit was chosen because the
SwiftUI↔AppKit seam caused recurring cursor/tooltip/resize glitches and a file-open crash. See
**FEATURES.md** for the per-feature log and **DECISIONS.md** for *why* each major choice was made
(read it before reworking an area; add an entry when you make a non-obvious design decision).

## Stack
- AppKit, built with Swift Package Manager (no Xcode; Command Line Tools only). Programmatic UI
  (no storyboards/xibs). Model layer uses **Combine** `@Published` (independent of SwiftUI).
- **SwiftTerm** — native terminal (ships its own PTY).
- **Editor highlighting is native** — a small TextMate-grammar engine driven by `NSRegularExpression`
  (`UI/Editor.swift` + `TextMate/TextMateHighlighter.swift`), no third-party dep. ~30 `.tmLanguage.json`
  grammars (from VS Code) live in `TextMate/Grammars/` and ship as the `Multee_Multee.bundle` SwiftPM
  resource. This replaced **Highlightr** (highlight.js via JavaScriptCore), which cost ~150 MB RAM per
  process for a ~5 MB on-disk JS bundle; the native engine is ~70% lighter on RAM at roughly the same
  app size. Grammars load lazily per language; "good, not tree-sitter-perfect" (regex, external-grammar
  includes skipped). See the resource-bundle gotcha below for how the bundle is resolved.

## Build & run
- `./dev.sh` — build (debug) → install to **`/Applications/Multee Dev.app`** → relaunch. Debug
  builds are a **separate app** ("Multee Dev", bundle id `com.multee.native.dev`, amber icon) so
  they never clash with a real/brew-installed Multee. Its defaults domain is `com.multee.native.dev`.
- `./build.sh release` — optimized **`Multee.app`** (`com.multee.native`). Copies the binary, icon,
  and SwiftPM resource bundles into `Contents/`, then ad-hoc signs.
- Type-check only: `swift build`.

## Releasing
- **The version is the git tag** — no version constant. `build.sh` reads `MULTEE_VERSION` (CI passes
  it from the tag, `v0.1.0` → `0.1.0`) into Info.plist.
- **Push a `v*` tag** → `.github/workflows/release.yml` builds the app, publishes the GitHub Release,
  and refreshes the Homebrew cask via the tap (`Rudra370/homebrew-tap`, needs `TAP_DEPLOY_KEY`).
- **Release notes are hybrid:** a `## [x.y.z]` section in `CHANGELOG.md` becomes the Release body and
  the in-app "What's new"; if absent, CI auto-generates from commits. Prefer writing the section.

## Debugging without a human (dev build only)
The dev build reads `/tmp/multee-debug.json` on launch (release ignores it):
```json
{ "shot": "/tmp/multee-shot.png", "state": "/tmp/multee-state.json",
  "actions": ["openRepo:/path", "openFile:rel", "openDiff:rel", "newClaude", "newTerminal",
              "closeActiveTab", "closeSession", "openSettings", "sendText:hi", "sendEnter",
              "scroll:up:10", "setStatus:needs", "editorType:x", "editorSave", "setFont:16",
              "editorFind:foo", "editorFindToggle:case|word|regex", "editorFindNext",
              "editorEol:CRLF", "editorIndent:Tabs", "editorLang:markdown", "paletteLineJump",
              "editorLineColors:1-100", "editorColorRuns:80,848",
              "gitCheckout:branch", "gitBranchNew:name", "gitBranchDel:name",
              "treeNewFile:a.txt", "treeNewFolder:dir", "treeBeginFile", "treeExpandAll",
              "treeCollapseAll", "treeRename:old.txt|new.txt", "treeDelete:path",
              "paletteOpen", "paletteCommands", "paletteType:foo", "paletteDown", "paletteUp",
              "paletteEnter", "paletteClose", "sidebarMode:2", "revealSearch", "projectSearch:foo",
              "searchOpenFirst", "searchOpenAsTab", "openSearchTab", "projectSearchTab:foo",
              "openAt:file.md|3", "setStatus:done", "hookStatus:0:idle", "activateTab:1",
              "renderAttentionMenu:/tmp/x.png", "showShortcuts", "renderShortcuts:/tmp/x.png",
              "quickToggle", "quickMode:floating|centered|bottom", "quickSend:echo hi",
              "quickNew", "quickActivate:1", "quickClose:1", "quickOpenAsTab",
              "newTermShortcut", "newClaudeShortcut", "newFile", "editorSaveAs:/tmp/x.md",
              "tabRestart", "tabToTerminal", "newProject:/tmp/x|git"] }
```
- `shot` → self-screenshot of the window each 1s (no Screen-Recording permission). **Captures
  standard AppKit (chips, tree, editor, diff, panels) but NOT the SwiftTerm terminal** — it draws via
  CoreText in a way `cacheDisplay` can't grab. **Verify terminal content via `terminalText` in the
  state dump, not the screenshot.** (Corollary: don't make the terminal's ancestor views non-layer
  to "fix" terminal capture — it breaks chip/editor capture instead. Terminal stays buffer-verified.)
- `state` → UI + active-terminal state each 1s (active session/tab, tab list, terminal rows/cols/
  scroll/repaints/**terminalText**, editorDirty, a `layout` frame diagnostic). Assert on values.
- `actions` → scripted with delays; `wait:N` inserts N extra seconds.
- `DebugHarness.swift` holds it all; `TerminalStore.debugText/debugState` inspect terminals.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rudra370/multee](https://github.com/Rudra370/multee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->

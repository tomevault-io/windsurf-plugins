---
trigger: always_on
description: A macOS Safari Web Extension that replicates the "Claude in Chrome" browser automation extension. It allows Claude Code CLI to control Safari via MCP (Model Context Protocol).
---

# Claude in Safari — Project Context

## What This Is
A macOS Safari Web Extension that replicates the "Claude in Chrome" browser automation extension. It allows Claude Code CLI to control Safari via MCP (Model Context Protocol).

## Quick Start
```
make dev                  # Build + launch + health check (full setup)
make test-all             # Run JS + Swift unit tests
make send TOOL=navigate ARGS='{"url":"https://example.com"}'
make doctor               # Full diagnostic if something seems off
```

## Architecture
- **Native Swift App** (`ClaudeInSafari/`): MCP socket server, screenshot capture, file I/O
- **Safari Web Extension** (`ClaudeInSafari Extension/`): Background script, content scripts, tool handlers
- Communication: CLI → Unix domain socket → Native App → `browser.runtime.sendNativeMessage()` → Extension → Content Scripts → Web Page

## Rules
- Always read PRINCIPLES.md before implementing any feature
- Always check STRUCTURE.md before creating or moving files
- Feature workflow: Spec → Test → Implement → Verify structure
- **One thing at a time**: always work on a single feature or fix per session; create a dedicated feature branch (`git checkout -b fix/...` or `feature/...`) before touching any code
- **Implementation plans** live in `docs/plans/` — one file per feature, named `YYYY-MM-DD-<feature>.md`
- **Version sync**: every PR must bump the version across all 3 sources (both `Info.plist`s + `manifest.json`). Use `scripts/bump-version.sh <new-version>`. CI enforces the match.

## Build After Changes
| What changed | Command |
|---|---|
| **Swift only** | `make kill && make build && make run` |
| **Any JS** (background.js, tool handlers, content scripts) | `make safari-restart` |
| **Both** | `make safari-restart` |

Safari caches background page JS — `make kill && make run` does NOT reload JavaScript. Only `make safari-restart` forces a JS reload (note: resets "Allow Unsigned Extensions").

## Testing
- `make test` — JS unit tests (tool handlers, content scripts). Fast, run after every JS change.
- `make test-swift` — Swift XCTests (MCP server, message framing, routing). Run after Swift changes.
- `make test-all` — Both. Run before every PR.
- `make functional-check` — End-to-end: sends a real `read_page` through the full pipeline. Requires running app + Safari.
- Manual regression: `docs/regression-tests.md` — required before merge (PRINCIPLES.md rule 8).

## Extension Not Loading?
1. `make health` → passes? You're fine.
2. Fails → `make kill && make build && make run && make health`
3. Still fails → Toggle extension off/on in Safari Settings, retry `make health`
4. Still fails → `make safari-restart` (resets Allow Unsigned Extensions — re-enable in Develop menu)
5. Still fails → `make doctor` for full diagnostics
6. See `docs/debugging.md` for the complete troubleshooting guide.

## Extension Workflow — Hard Rules
- **Never run `xcodebuild clean` alone.** The first build after a clean produces an invalid app signature, causing pluginkit to silently drop the extension. Always use `make clean` (which runs `clean build` in one invocation) or just `make build`.
- **Never use `pluginkit -e use/ignore`.** Force-overriding pluginkit state conflicts with Safari's native extension management. Use `pluginkit -e default` to reset, or don't touch pluginkit at all.
- **Always use `make kill`** to stop the app — Xcode's debugserver can hold zombie processes in `TX` (stopped) state, blocking extension loading.

## Safari Pitfalls
These platform quirks affect implementation decisions across the project:
- **`executeScript` requires Safari frontmost** — fails silently otherwise. Use `activateSafariIfNeeded()` (Spec 024).
- **`browser.tabs.query` returns empty** inside native messaging handlers — needs `setTimeout(0)` dispatch + retry loop.
- **Integer `1` arrives as boolean `true`** via the Swift native bridge (NSNumber/JSON serialization) — always use explicit type checks.
- **BFCache skips `"loading"` events** — `goBack()`/`goForward()` may jump straight to `"complete"`. Don't require `"loading"` before accepting `"complete"` for history navigations.
- **`browser.storage.session` resets** if background page suspends (mitigated by `persistent: true`).
- **`read_page` refs ≠ `computer` refs** — accessibility tree uses WeakRef map (`__claudeElementMap`); `computer.js` uses `data-claude-ref` DOM attrs (set only by `find.js`). Must use `find` before `computer(ref)`.

## Key Technical Decisions
- **MV2 manifest** with `"persistent": true` — MV2 avoids MV3's service-worker lifecycle unpredictability on macOS Safari; `persistent: true` is required on Safari 26+ because the background page never bootstraps with `false` (the event that would wake it never fires, since polling is initiated from the background itself)
- **ScreenCaptureKit** for screenshots (Safari's `captureVisibleTab` is unreliable)
- **App Sandbox** enabled for App Store distribution; file access uses security-scoped bookmarks via `FileAccessManager.swift`
- **Virtual tab groups** via `browser.storage.session` (no `browser.tabGroups` API in Safari)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chriscantu/claude-safari-extension](https://github.com/chriscantu/claude-safari-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->

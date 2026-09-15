---
trigger: always_on
description: Carrier is a tiny **Tauri v2** desktop client for **Facebook Messenger** (wraps
---

# Carrier — project guide & agent handoff

Carrier is a tiny **Tauri v2** desktop client for **Facebook Messenger** (wraps
`facebook.com/messages` in a chrome-stripped native window). Repo:
**`kristofferR/Carrier`** (use the **kristofferR** GitHub account). **v1.0.0 is
released.** **Cross-platform: macOS, Windows, and Linux** (CI builds all six
targets). The "macOS theme rendering" section below is platform-specific; on
Windows/Linux the window chrome follows `set_theme` directly and there's none of
the WKWebView/title-bar trouble.

---

## Build / run / install

```bash
# from repo root
bun install
bun run tauri build               # release-ish bundle (debug symbols unless --release config)
bun run tauri build --debug --bundles app   # fast debug .app only

# signed macOS release (Developer ID) — for the real install:
export APPLE_SIGNING_IDENTITY="Developer ID Application: Kristoffer Risanger (S5Q742QZEL)"
bun run tauri build               # uses src-tauri/tauri.conf.release.json via the CI flow

# install without a Gatekeeper prompt (rm -rf on /Applications is blocked by the safety net):
ditto "<path>/Carrier.app" "/Applications/Carrier.app"
xattr -dr com.apple.quarantine "/Applications/Carrier.app"
```

**Gate before every commit** (CI mirrors this):
```bash
cd src-tauri && cargo fmt && cargo clippy --all-targets -- -D warnings && cargo test --lib
bun run check   # Biome lint + tsc typecheck + bun test + rebuild the inject bundles
```

The injected scripts are TypeScript now: **edit `inject/src/`, never
`src-tauri/inject/*.js`** (generated; `messenger.css` and the dev-only
`mcp-bridge.js` are still hand-maintained). After changing `inject/src/`, run
`bun run build:inject` and commit the regenerated bundles alongside the source
— CI fails if they drift. Pure logic lives in `inject/src/messenger/lib/` with
`bun test` coverage; add tests there when extending it.

Releases use curated notes, not a generator. Before pushing a `v*` tag, inspect
the exact tag range and merged PRs and write the release title/body with human or
agent judgment, using direct sections such as **What's New**, **Bug Fixes**, and
**Internal** only when useful. Create the GitHub draft first, for example:

```bash
gh release create v1.4.0 --draft --target main --title v1.4.0 --notes-file release-notes.md
git tag v1.4.0 <release-commit>
git push origin v1.4.0
```

`.github/workflows/release.yml` requires a non-empty curated draft, builds 6
targets, signs + notarizes macOS, attaches artifacts without rewriting the title
or body, and publishes it. Apple/Tauri secrets are set in the repo.

---

## tauri-mcp — dev webview inspection

Lets an agent inspect/drive the running webview: `execute_js`, `query_page`
(DOM), **`take_screenshot` (background — no window popping to the foreground)**,
click/type, etc. Plugin: [P3GLEG/tauri-plugin-mcp](https://github.com/P3GLEG/tauri-plugin-mcp).
On `main` (committed `d8a25a6`).

- Gated behind the Cargo **`mcp` feature** → **release builds never compile it**.
- Build with it: `bun run tauri build --debug --features mcp --bundles app`. Debug
  builds are marked — the window title reads **"Carrier (debug)"**.
- `.mcp.json` registers the `tauri-mcp` server — **approve the project MCP server
  when a new session prompts for it.**
- **A "Carrier (debug)" build must be RUNNING** for the tools to connect — build it
  with the command above, then `ditto` it into `/Applications`.
- For Hide Names & Avatars selector work, first run the dev-only sanitized probe
  through MCP `execute_js` with code `__carrier_mcp_privacy_probe__`. It reports
  live Messenger selector hits, rectangles, computed `filter`, and sanitized
  ancestor/attribute shapes; it must not expose message text, raw numeric IDs,
  image URLs, or `alt` / `aria-label` contents. Prefer extending this probe over
  guessing selectors or dumping raw DOM when debugging privacy blur coverage.
- Notification work has the same treatment: `__carrier_mcp_notification_row_probe__`
  (conversation-row and message-row shape — where emoji sprites sit, which
  images a row carries) and `__carrier_mcp_sender_avatar_probe__` (what the
  sender-avatar harvest would take, what it cached, and whether each visible
  group preview's sender resolves to a face). Both report classifications and
  counts only.

---

## Architecture

- **`src-tauri/src/lib.rs`** — the Rust shell: window/tray/menu/settings/theme,
  `on_navigation` (off-site → default browser), `on_download` (media only, blocks
  executables), updater.
- **`src-tauri/inject/`** — injected at document-start: `messenger.css` (hides FB
  chrome + theme/compact/login CSS), `messenger.js` (shortcuts, zoom, image
  viewer, notifications incl. mute / hide-preview privacy, unread badge,
  force-theme, hide names & avatars, login tidy), `panel.js` (toast,
  settings/update bridge). `messenger.js`/`panel.js` are **generated** — the
  typed sources live in **`inject/src/`** (one module per feature; pure,
  unit-tested logic under `inject/src/messenger/lib/`), bundled back to single
  IIFE files by `bun run build:inject` (esbuild, config in `inject/build.ts`).
  Init order in `inject/src/messenger/index.ts` mirrors the old single-file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kristofferR/Carrier](https://github.com/kristofferR/Carrier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->

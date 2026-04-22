---
trigger: always_on
description: - The active local/debugging build surface is the generated `dist/` directory.
---

# AGENTS.md

## Debug Workflow

## Deployment Note

- The active local/debugging build surface is the generated `dist/` directory.
- Keep deployment assumptions aligned with the current repo contents.

### Fixed Tampermonkey / Gemini Environment

- Fixed Chrome profile: `.chrome-debug/tampermonkey-profile`
- Fixed CDP port: `9226`
- Default proxy: `http://127.0.0.1:7890`
- Production userscript artifact: `dist/userscript/gemini-watermark-remover.user.js`

Platform notes:

- macOS launcher uses `/usr/bin/open -na "Google Chrome.app" --args ...`
- macOS default Chrome app lookup:
  - `/Applications/Google Chrome.app`
  - `~/Applications/Google Chrome.app`
- Override Chrome location on any platform with `GWR_DEBUG_EXECUTABLE_PATH`

### Open the Fixed Profile

- Shell launcher: `./scripts/open-fixed-chrome-profile.sh`
- Node launcher: `node scripts/open-tampermonkey-profile.js --cdp-port 9226`
- CMD launcher: `.\scripts\open-fixed-chrome-profile.cmd`
- PowerShell launcher: `.\scripts\open-fixed-chrome-profile.ps1`

Default behavior:

- Reuse the fixed Chrome profile
- Open remote debugging on port `9226`
- Use the local proxy
- The plain Node launcher opens the Tampermonkey Chrome Web Store page by default unless `--url` is passed
- The shell wrapper opens the local probe page on `http://127.0.0.1:4173/tampermonkey-worker-probe.html` by default and still forwards extra args
- The current CMD/PowerShell wrappers are fixed shortcuts to the same local probe page; they do not forward additional CLI args

macOS quick path:

1. Run `pnpm build`
2. Start the local artifact server, for example `pnpm dev`
3. Open the fixed profile with `./scripts/open-fixed-chrome-profile.sh --url https://gemini.google.com/app`
4. If you do not want the repo's default proxy, pass `--proxy off`

### One-Time Manual Setup

Do this only once in the fixed profile:

1. Install Tampermonkey.
2. Enable `Allow User Scripts` in Chrome extension details.
3. Keep Developer Mode enabled.
4. Install `public/tampermonkey-worker-probe.user.js` when local probe validation is needed.
5. Install or reinstall the production userscript from the current local build server when validating the latest build.
   - Use the active local build server URL printed by `pnpm dev`
   - `pnpm dev` starts probing from `http://127.0.0.1:4173/` and will auto-increment if that port is occupied
   - A previously confirmed request-layer debugging session used `http://127.0.0.1:4317/userscript/gemini-watermark-remover.user.js`, but do not assume that is still current

### Local Build and Services

- Production build: `pnpm build`
- Local dist server: `pnpm dev`, `pnpm serve`, or the active local build server for this worktree
- Default `pnpm dev` start port: `http://127.0.0.1:4173/`
- Actual active dev server port may be higher; always trust the current `pnpm dev` console output
- Probe smoke test: `pnpm probe:tm`
- Installed userscript freshness check: `pnpm probe:tm:freshness`
- Open fixed profile: `pnpm probe:tm:profile`

Current `pnpm probe:tm` behavior:

- in `run` mode it now attempts a Tampermonkey userscript freshness preflight first
- if freshness returns `stale`, `probe:tm` must fail before running the worker/bridge smoke page
- if the freshness preflight context itself is unavailable, for example:
  - fixed `9226` profile is not open
  - the CDP endpoint is unavailable
  - the editor is mid-navigation
  then the preflight is recorded as `skipped` and the smoke flow continues
- this keeps stale installs fail-fast without making `probe:tm` hard-depend on a manually opened editor page

### Installed Userscript Freshness Check

When real-page behavior does not match the current worktree, verify the installed userscript body, not just the script name or `@version`.

- Same `@version` does not guarantee the fixed profile is running the latest build.
- A stale Tampermonkey script can still show:
  - `[Gemini Watermark Remover] Initializing...`
  - `[Gemini Watermark Remover] Ready`
  - while silently missing newer request-layer fixes such as the download sticky intent window
- Preferred check:
  1. Open the Tampermonkey editor for `Gemini NanoBanana 图片水印移除`
  2. Run `pnpm probe:tm:freshness`
  3. Read `.artifacts/tampermonkey-freshness/latest.json`
- If you just reinstalled the userscript and the check still reports `stale`, refresh the already-open Tampermonkey editor page once, then run `pnpm probe:tm:freshness` again.
- Current command behavior:
  - exits `0` when the installed userscript exactly matches the local `dist/userscript/gemini-watermark-remover.user.js`
  - exits `1` when the installed userscript is stale or mismatched
  - compares full normalized source hashes, not just `@version`
  - also reports whether expected markers are missing
- Current report path:
  - `.artifacts/tampermonkey-freshness/latest.json`
- Manual fallback if needed:
  1. Compare the installed source against `dist/userscript/gemini-watermark-remover.user.js`
  2. Confirm the installed source contains the expected newer markers before continuing real-page debugging
     - `DEFAULT_DOWNLOAD_STICKY_WINDOW_MS`
     - `downloadStickyUntil`
     - `getActionContextFromIntentGate(intentGate = null, candidate = null)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GargantuaX/gemini-watermark-remover](https://github.com/GargantuaX/gemini-watermark-remover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

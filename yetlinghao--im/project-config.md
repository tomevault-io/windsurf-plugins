---
trigger: always_on
description: Tauri 2 app (Rust backend, vanilla TypeScript + Vite frontend), macOS first.
---

# im — working notes for agents

Tauri 2 app (Rust backend, vanilla TypeScript + Vite frontend), macOS first.
README.md is the front door and stays a few lines — the user wants it to read
like the landing page. docs/DEVELOPMENT.md has the build/usage docs, the
on-disk schema, the protocol table and the release procedure.

## Build & verify

```sh
npm install
npx tsc --noEmit                              # frontend typecheck
(cd src-tauri && cargo test)                  # 41 tests: parsers, store, engine e2e over a local SSE server
npx tauri build --debug --bundles app         # debug .app → src-tauri/target/debug/bundle/macos/im.app
npm run dev &  scripts/snapshot.sh            # frontend states via headless Chrome → build/snapshots/*.png
scripts/app-snapshot.sh out.png [light|dark]  # the REAL app renders its own window to PNG
swiftc -O -o build/icon scripts/icon.swift && build/icon logo.PNG src-tauri/icons/source.png \
  && npx tauri icon src-tauri/icons/source.png -o /tmp/icons  # app icon from the logo; copy the mac/win files back
npm run tauri build                           # release .app + updater archive (ad-hoc signed; no dmg by design)
git tag vX.Y.Z && git push origin vX.Y.Z      # GitHub Actions builds, signs the updater archive and publishes the Release
```

Releases live at github.com/yetlinghao/im; the landing page is `site/` →
GitHub Pages at im.linghaoz.com (`.github/workflows/pages.yml`, custom domain set
via the Pages API; DNS is a CNAME to yetlinghao.github.io). `install.sh` (repo
root, copied into the site at deploy) is the primary install path: no
quarantine flag → no Gatekeeper. The page is deliberately just icon, slogan,
one line, the command, and one figure (`.why`: im / Cherry Studio / ChatGPT as
three rows × two bar columns, "Download" 5 MB · 500 MB · 1.39 GB on a linear
scale and "What it covers" ~90 · 95 · 100 %, the extra slice in a lighter gray,
two legend lines naming the everyday vs. the other 10%) — the user vetoed
screenshots, a dmg link and a grow-in animation on the bars; plain CSS, no JS.
Headless Chrome won't go below ~500px wide, so check the mobile layout at 500,
not 390. The updater
(`tauri-plugin-updater`) polls `releases/latest/download/latest.json`; its
public key is in `tauri.conf.json`, the private key is `~/.tauri/im.key` on
the user's Mac and the `TAURI_SIGNING_PRIVATE_KEY` repo secret — never in the
repo. Frontend: `actions.checkForUpdates/installUpdate`, `state.update`; the only
unprompted surface is one line at the foot of the sidebar (`.update-row`,
"Update to 0.2.0" → "Downloading… 42%" → relaunch) that exists only while an
update is waiting; Settings → General → Version and `im → Check for Updates…`
are the manual paths. No dialogs, no badges. There is no dmg any more
(`bundle.targets = ["app"]`): install.sh is the only install path, by the
user's decision. Mock: `?update=1` fakes a 0.2.0 in the feed.

This terminal cannot take screenshots or send keystrokes. Two ways to see the
UI, use both after any view change:

- `scripts/snapshot.sh` runs the frontend against the in-memory mock backend
  (`src/api.ts`, active whenever `__TAURI_INTERNALS__` is missing) in headless
  Chrome. `?state=chat|streaming|picker|settings|empty|noproviders|error|edit|nosidebar|json|scrolled|resized|html|html-expanded|streaming-html|svg|image|image-expanded|attach|select-test`
  (`quick.html?state=empty|typed|long` is the quick-input panel's page)
  (iframes need `--virtual-time-budget=3000` on the shot to have loaded)
  and `?theme=light|dark` pick the scenario; `&inspector=1` opens the right
  column (`collapse` toggles the sidebar in 4s slow motion so a
  snapshot catches the slide; `collapse-frames` runs it at real speed and logs
  the frame count; `resized` drags both column handles synthetically). Chrome doesn't exit cleanly in
  this sandbox, so `scripts/shot.sh` polls for the file and kills it. Requests
  need `--proxy-server=direct://` because the shell has an HTTP proxy set.
- `scripts/app-snapshot.sh` launches the debug bundle with fixture data
  (`scripts/fixtures/`; `SIDEBAR=0` closes the sidebar, `INSPECTOR=1` opens the right column) and env `IM_SNAPSHOT_PATH`; `src-tauri/src/snapshot.rs`
  (debug builds only) captures the window via `CGWindowListCreateImage`
  (dlsym'd — deprecated but works for our own window) including what is on
  screen below it, so vibrancy shows what it really blurs. `IM_SCENARIO=` and
  `IM_AUTOSEND=` drive the UI (`debug_scenario` command); `PROVIDER=mock`
  points the fixtures at `scripts/mock_server.py` (start it first) for a real
  end-to-end turn — its log prints one line per request (`messages=[u72 a879+r …]`,
  `+r` = that reply was replayed with `reasoning_content`); `KEEP_DATA=1` keeps
  the temp data dir to inspect the JSON. The app is brought to the front before
  the capture (`ACTIVATE=0` to see the inactive look) — it steals focus, so
  don't type while a snapshot runs.
- Webview `console.error/warn` and uncaught errors are forwarded to the Rust
  log (`log_message` command); run with `RUST_LOG=im_lib=debug,webview=debug`
  (the crate is `im_lib`, so `im=…` matches only by prefix).

## Architecture

```
src-tauri/src/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yetlinghao/im](https://github.com/yetlinghao/im) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->

---
trigger: always_on
description: WebSpace: Flutter app managing multiple websites with per-site cookie isolation via flutter_inappwebview. Platforms: iOS, Android, macOS, Linux (WPE WebKit fork).
---

# CLAUDE.md

WebSpace: Flutter app managing multiple websites with per-site cookie isolation via flutter_inappwebview. Platforms: iOS, Android, macOS, Linux (WPE WebKit fork).

## Style (output, code, commits)

- No preamble, no closing fluff, no em-dashes, no emoji.
- Code first; explain only the non-obvious.
- Default to **no code comments**. Only add when the *why* is non-obvious (hidden constraint, workaround for a specific bug, surprising behavior). Never restate what the code does. Never reference the current task or PR.
- Commit messages: short subject (<70 chars, imperative), 1-2 line body for the *why* if needed. No marketing prose, no bullet lists of every changed file, no "this commit also...".
- Don't speculate. Read the code or docs before asserting an API/version/flag.

## Git

- Never push to master. Branch first.
- `git pull --rebase` always.
- Before pushing a branch, check it still exists on remote; if merged+deleted, branch fresh from master.
- **Don't commit derivatives.** If a file is the output of a script, parser, compiler, dumper, or any build step that reads from elsewhere — it doesn't belong in the repo. Commit the inputs (sources you author, pinned upstream refs) and the *runner* (build.rs, scripts, Cargo features); regenerate the output at build time into `$OUT_DIR`/`build/`/`target/`. Same applies to vendored third-party source: if a script can fetch + assemble it from upstream at a pinned ref, don't check the upstream tree in. Concrete check before staging: "could I delete this file and reproduce it by running one command from a clean clone?" — if yes, it's a derivative; ignore it. See `rust/webspace_adblock/build.rs` for an example.

## Sandbox bootstrap

Fresh sandboxes lack `fvm` and may lack `nvm`/Node. Skip a block if `command -v fvm` (or `node`) already prints a path — don't reinstall.

```bash
# fvm — required (.fvmrc pins Flutter 3.38.6)
curl -fsSL https://fvm.app/install.sh | bash
export PATH="$HOME/fvm/bin:$PATH"
fvm install

# nvm + Node — only if a script under scripts/ or tool/ needs Node
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
export NVM_DIR="$HOME/.nvm"; [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install --lts
```

## Commands

Prefix all Flutter/Dart with `fvm`.

```bash
fvm flutter pub get
fvm flutter test                                         # all Dart tests
fvm flutter test test/cookie_isolation_test.dart         # single file
npm run test:js                                          # JS shim tests (jsdom)
./scripts/test_all.sh                                    # Dart + JS
fvm flutter analyze
fvm flutter build apk    --flavor fdroid --release       # F-Droid (CI, unsigned)
fvm flutter build apk    --flavor fmain  --release --split-per-abi   # Play (signed)
fvm flutter build ipa    --release                       # iOS unsigned
fvm flutter build macos  --release
fvm flutter build linux  --release
fvm dart run flutter_launcher_icons
```

Android flavors: `fdroid` (CI), `fmain` (Play), `fdebug`.

## Architecture

**Models**
- `WebViewModel` ([lib/web_view_model.dart](lib/web_view_model.dart)) — site with URL, cookies, per-site settings (language, incognito, proxy, etc.). Unique `siteId` keys cookie isolation.
- `Webspace` ([lib/webspace_model.dart](lib/webspace_model.dart)) — named collection of site indices. `__all_webspace__` shows all.

**Main** — [lib/main.dart](lib/main.dart): `WebSpaceApp` (root MaterialApp) and `WebSpacePage` holding `_webViewModels`, `_webspaces`, `_loadedIndices`, isolation orchestration.

**Services** ([lib/services/](lib/services/)) — `cookie_secure_storage`, `html_cache_service` (AES, clears on upgrade), `icon_service`, `dns_block_service`, `webview` (CookieManager wrapper, WebViewTheme).

**Cookie isolation — two engines, runtime-selected.** `_WebSpacePageState` caches `bool _useContainers = await ContainerNative.isSupported()` at startup and gates the path.

- **Container engine** ([container_isolation_engine.dart](lib/services/container_isolation_engine.dart)) — Android System WebView 110+, iOS 17+, macOS 14+, Linux WPE WebKit 2.40+. Each `siteId` → native container `ws-<siteId>` (`androidx.webkit.Profile` / `WKWebsiteDataStore(forIdentifier:)` / `WebKitNetworkSession` cached under `<XDG_DATA_HOME>/flutter_inappwebview/containers/`) owning its cookies, localStorage, IDB, ServiceWorkers, HTTP cache. Same-base-domain sites load concurrently — no conflict-unload, no capture-nuke-restore. Bridge: [`ContainerNative`](lib/services/container_native.dart). Lifecycle ops route through fork's `inapp.ContainerController`; only the Android `MULTI_PROFILE` feature gate lives in [`WebSpaceContainerPlugin.kt`](android/app/src/main/kotlin/org/codeberg/theoden8/webspace/WebSpaceContainerPlugin.kt). Bind happens in `InAppWebView.prepare()` / `preWKWebViewConfiguration` / Linux `webkit_web_view_set_property("network-session", ...)`, driven by stock `inapp.InAppWebViewSettings.containerId` set by `WebViewFactory.createWebView`. Spec: [openspec/specs/per-site-containers/spec.md](openspec/specs/per-site-containers/spec.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theoden8/webspace_app](https://github.com/theoden8/webspace_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

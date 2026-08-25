---
trigger: always_on
description: Electron shell around the hosted web app. It is a **remote-URL wrapper**: the package
---

# langalpha desktop

Electron shell around the hosted web app. It is a **remote-URL wrapper**: the package
contains Electron, `src/`, two local pages and an entry URL, and nothing else. No web
bundle ships with it, so a web deploy never needs a desktop release. The two local
pages are the ones that have to work when the network does not: the OSS server picker
and the outage screen.

> Single source of truth for AI coding agents in `desktop/`. Edit here.

## Why Electron, not Tauri

Both were built and measured. On macOS the performance is comparable, so speed is not
what decided it: what is left is a straight trade of package size and idle memory, where
Tauri wins outright, against engine control and shell cost, where Electron does.

**Engine control is worth more here than it would be in a bundled app.** This wrapper
carries no web bundle, so under Tauri the rendering engine becomes whatever the user's
OS ships. WebView2 on Windows is Chromium and fine; macOS pins to that machine's
WKWebView and Linux to WebKitGTK, for a payload that deploys weekly and cannot be
version-matched to any of them. Electron puts one Chromium inside the package and moves
it on our schedule. For an app that shipped its own bundle this argument would be much
weaker, because engine and payload would be tested as a pair.

**The window chrome is the same point in miniature.** `-webkit-app-region` is a Chromium
property and the whole contract is built on it, five call sites in `web/` plus
`titleBarStyle`. Tauri's `data-tauri-drag-region` would put shell-specific markup into
`web/` for every browser user to download, and put a JS handler back in front of a
decision that is now pure CSS.

**The shell is cheap because it is JavaScript.** Roughly 2000 lines of URL classification
and window policy, with a suite in plain `node --test` over an `electron` stub and no
runtime needed. In Rust that is a toolchain in CI and on every self-hoster's machine, to
host logic that mostly decides whether a URL is ours.

This is the right call for this stage, not a permanent one. The thing that would move it
is the desktop app shipping its own bundle: engine and payload recouple, the argument
above loses most of its force, and ~90 MB of package and the idle memory become the
whole story.

## Commands

```bash
pnpm start                       # run the shell from source (oss defaults → localhost:5173)
pnpm test                        # node:test; pure logic only, no Electron runtime needed
pnpm run preview                 # the shell against a web build that has not deployed yet
pnpm run build                   # unpacked build into dist/<edition>/, fastest way to check packaging
pnpm run dist                    # real installers (dmg + zip on macOS)

DESKTOP_EDITION=saas \
DESKTOP_APP_ORIGIN=https://…  \
DESKTOP_PLATFORM_ORIGIN=https://… pnpm run dist  # a saas package
```

**`pnpm run preview` is how you see a change before it deploys.** A remote-URL shell
loads whatever is *live*, so every frame decision it makes is a reaction to the deployed
bundle rather than to the source in front of you: that is how the window buttons ended up
on the app's own logo, and it is where the console's second window was caught before it
reached anyone. It builds `web/`, serves it on loopback with the api streamed through to
a running stack, and launches the shell against it. `--web-env <file>` builds the
frontend the way another environment builds it and `--platform <origin>` runs the saas
edition, which together make a hosted preview a matter of pointing at a local pair:

```bash
# the console, in the layout production actually runs: at an origin root, not
# under /account/. Its dev default is the legacy same-host layout, and a console
# reachable only at a path prefix is one this shell cannot address, since it
# classifies by origin and nothing else.
VITE_ACCOUNT_PREFIX= VITE_BASE=/ VITE_APP_URL=http://localhost:5399 \
VITE_PLATFORM_URL=http://localhost:5178 VITE_COOKIE_DOMAIN=localhost \
pnpm dev --port 5178 --strictPort          # in the console's own web/

VITE_APP_ENTRY_PATH=/ VITE_PLATFORM_URL=http://localhost:5178 \
pnpm run preview -- --host localhost --web-env ../../web/.env \
  --platform http://localhost:5178 --backend http://127.0.0.1:8000
```

`--host localhost` rather than the loopback literal is deliberate: cookies ignore
port, so a console on the same hostname shares a session with the preview the way
the two subdomains share one in production.

It keeps a user-data dir of its own, which is not tidiness: sharing the installed app's
would teach the *installed* app that its frontend reserves the window-button strip when
the build it actually loads may not, which is the bug it exists to catch.

**Say `pnpm run`, not `pnpm`, for anything that builds.** `pnpm <name>` falls back
to a script only when pnpm has no command of that name, and the failure when it
does is silent: this used to be `pnpm pack`, which quietly built a **tarball**
instead, left the previous artifact in the output tree, and exited 0. A verification run
against that stale artifact is what caught it.

**The `postinstall` line is load-bearing.** Electron 42 removed the package's own

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ginlix-ai/LangAlpha](https://github.com/ginlix-ai/LangAlpha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->

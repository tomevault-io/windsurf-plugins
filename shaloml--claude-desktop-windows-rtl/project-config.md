---
trigger: always_on
description: In-place patchers that inject RTL + extensions into the **official Claude
---

# Claude Desktop — Windows, macOS & Linux RTL & extensions patch — dev notes

In-place patchers that inject RTL + extensions into the **official Claude
Desktop**: Windows (Microsoft Store / MSIX build) via PowerShell, macOS
(`/Applications/Claude.app`) via a bash script, and Linux
(`/usr/lib/claude-desktop`, claude-desktop-debian layout) via a bash script.
Read this before making non-trivial changes.

## Platforms

- **Windows:** `patch-claude-windows.ps1` + `src/win-entry.js` / `win-wrapper.js`.
  Verified working.
- **macOS:** `patch-claude-macos.sh` + `src/mac-entry.js` / `mac-wrapper.js`.
  Verified on macOS 26 (Apple Silicon). Differences from Windows: `sudo` instead
  of takeown/icacls (skipped when the user owns the bundle); asar integrity
  updated in `Info.plist` (`ElectronAsarIntegrity`) instead of byte-replacing a
  binary; `codesign --force --deep --sign -` (ad-hoc) + `xattr -dr
  com.apple.quarantine` instead of a self-signed cert; launchd LaunchAgent
  instead of a Scheduled Task. Two verified consequences of the ad-hoc re-sign:
  (1) the app loses access to the `Claude Safe Storage` keychain key, so the
  first launch after each patch logs the user out once (re-login then sticks
  across normal restarts); (2) Claude's Squirrel updater rejects its own
  downloads, so a patched app won't auto-update. `asar pack` MUST pass
  `--unpack "{*.node,spawn-helper}"` or the native modules get packed into the
  asar and the app crashes at startup.
- **Linux:** `patch-claude-linux.sh` + `src/linux-entry.js` / `linux-wrapper.js`.
  **Upstream renamed itself `claude-desktop-debian` → `claude-desktop-unofficial`
  (v1.20186.1, Jul 2026) and that rename is NOT cosmetic — see the "Linux: the
  -unofficial rename" section below for what actually changed.** Everything the
  patcher touches is now resolved at runtime (install dir, launcher, Electron
  binary), so it patches both layouts; never re-hardcode `/usr/lib/claude-desktop`.
  By far the simplest target: nothing validates the asar on Linux and nothing is
  signed, so there is NO hash to patch and NO re-signing — it's the macOS flow
  minus Phase 2 (integrity) and Phase 3 (codesign). App files under the install dir
  (`/usr/lib/claude-desktop-unofficial`) are root-owned,
  so privileged writes go through per-operation `sudo` — run as your NORMAL user
  (not under sudo) so `npx`/Node stay on PATH; the heavy lifting (asar
  extract/pack) runs as you and only the writes into the install dir elevate.
  Two Linux-specific gotchas, both found the hard way: (1) `asar pack` MUST use
  `--unpack "{**/*.node,**/spawn-helper}"` — the natives are NESTED
  (`node_modules/.../*.node`), so a bare `*.node` (what the Win/mac globs use)
  matches nothing and silently packs them in, crashing startup; the upstream
  build packs with `**/*.node`. (2) `asar extract` reads the sibling
  `app.asar.unpacked/` dir, so ALWAYS extract from the real install path — never
  a lone copy of `app.asar` (extract then dies ENOENT on the natives, and the
  "Node.js vX" line printed is a crash banner, not info). Auto-re-patch is a
  systemd SYSTEM timer (`claude-linux-rtl.timer`, root, boot + every 3h) that
  re-applies from a stable copy when the installed `app.asar`'s SHA-256 differs
  from the recorded patched SHA — version-agnostic and needs no Node in the
  watcher (state is a sourceable `state.env`, not JSON). **"New window" — TWO
  flavors, because the app is one-window-per-profile.** `app.on('second-instance')`
  only focuses the existing window, and an externally-created `BrowserWindow` is
  "unknown" to the app's internal `<Window>` pool — so MCP connectors only wire
  into windows the app itself manages. There is no single window that has BOTH
  shared Cowork AND connectors; `linux-wrapper.js` exposes both as menu items: (1)
  IN-PROCESS `openNewWindow()` — a `new BrowserWindow` in this process sharing the
  session/user-data-dir → shared Cowork history, but no MCP connectors; (2)
  SEPARATE INSTANCE `multiInstance.openNewInstance()` — the launcher's
  `--new-window`, a separate `Claude-instance-N` profile the app fully manages →
  MCP connectors work, Cowork starts blank; **LEGACY ONLY — gone in
  `claude-desktop-unofficial`, so `supportsNewInstance()` probes the launcher text
  and `linux-wrapper.js` hides the menu item when it is absent.** The floating
  button uses #1. Both are debounced (~1.5s). The in-process button emits a PRIVATE trigger
  (`NEW_WINDOW_TRIGGER`), NOT the shared `multi-instance-support.js`
  `CONSOLE_TRIGGER`, so the host `frame-fix-wrapper`'s bridge doesn't ALSO spawn a
  separate process (that double-fire was the earlier "new window closes the other"
  bug). (Reusing the content view's `preload` via `getLastWebPreferences()` was
  tried to get MCP into the in-process window — it did NOT work; the app gates MCP
  on its own window management, not just the preload.) We do NOT defer to the host
  (an earlier deferral removed the RTL/button that only OUR wrapper reliably
  renders on this build). **Failure-safe + upgrade-safe install:** the patcher
  extracts the CURRENT live asar into a temp dir, injects, repacks, and writes the
  live asar exactly ONCE at the end — so a mid-run failure (npx hiccup, declined

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shaloml/claude-desktop-windows-rtl](https://github.com/shaloml/claude-desktop-windows-rtl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

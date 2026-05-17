---
trigger: always_on
description: Every desktop repackage must preserve the Linux `/pet` overlay customizations in `src/repack.js`.
---

# Codex Linux App Agent Notes

## Pet Overlay Defaults

Every desktop repackage must preserve the Linux `/pet` overlay customizations in `src/repack.js`.

- Keep the pixel pet usage UI as a yapping speech bubble, not neon rings.
- The bubble polls Codex usage every 10 seconds through the main-process app bridge.
- Bubble text must stay in English:
  - `5-hour usage left: N%`
  - `Weekly usage left: N%`
- The hover info under the pet must show both values in one pixel square:
  - `5H left N% | Weekly left N%`
- The bubble should be pixelated, compact, close to the pet head, and should fade out instead of disappearing instantly.
- Keep the transparent overlay footprint only as large as needed for the pet, bubble, and hover info. `.codex-usage-yap-wrap` is intentionally used as the avatar layout measurement target so the speech bubble and hover info are not clipped, but do not add extra unused transparent padding beyond those visible elements.
- Do not add a glow ring, aura, halo, or external usage ring around the pet unless the user explicitly asks to bring that design back.
- If the yapping bubble disappears after an upstream update, inspect the installed `webview/assets/avatar-overlay-page-*.js`, not only the unit fixture. The injection must use the real JSX runtime from the `jsx-runtime` import and attach to the mascot hit-region (`data-avatar-overlay-hit-region="mascot"`), not to notification scroll controls or the React compiler cache helper.
- If the yapping bubble stays on `Checking usage...`, do not import a minified `codex-api` export as the usage fetcher; in 26.513 `codex-api` export `n` was a worktree upload helper, not usage. The renderer must call the VS Code bridge handler `codex-linux-pet-usage`, and the main-process provider must read the latest `payload.rate_limits` from `~/.codex/sessions/**/*.jsonl`.
- Installed-bundle verification should confirm `codexLinuxPetYappingUsage`, `codexLinuxFetchUsage`, `.codex-usage-yap-wrap`, `codex-linux-pet-usage`, and the mascot hit-region are all present together, and that stale `codexLinuxUseQuery` is absent.

Reference implementation:

- Main usage provider patch: `patchMainProcessLinuxPetYappingUsage`, `injectLinuxPetYappingUsageMainPatch`, and `buildLinuxPetYappingUsageMainHandler` in `src/repack.js`.
- Renderer JS patch: `patchRendererLinuxPetYappingUsage`, `injectLinuxPetYappingUsagePatch`, and `buildLinuxPetYappingUsageComponent` in `src/repack.js`.
- Renderer CSS patch: `injectLinuxPetYappingUsageCssPatch` and `buildLinuxPetYappingUsageCss` in `src/repack.js`.
- Tests: `injectLinuxPetYappingUsagePatch adds yapping usage bubble to avatar overlay renderer` and `injectLinuxPetYappingUsageCssPatch adds pixel yapping styles` in `test/repack.test.js`.

Reference behavior shape:

```js
codexLinuxFetchUsage(`codex-linux-pet-usage`);
```

```css
.codex-usage-yap-pop {
  image-rendering: pixelated;
  animation: codex-usage-yap-pop 10s ease-in-out both;
}

@keyframes codex-usage-yap-pop {
  0% { opacity: 0; transform: translateY(8px) scale(.9); }
  3%, 24% { opacity: 1; transform: translateY(0) scale(1); }
  34%, 100% { opacity: 0; transform: translateY(-4px) scale(.96); }
}
```

## Pet Overlay Close Behavior

The avatar overlay must not keep Codex alive after the main Codex window closes.

- Keep `codexLinuxRegisterAvatarOverlayAutoClose` and `codexLinuxCloseAvatarOverlayIfOnlyWindow` in the main bundle patch.
- Listen to `closed` on existing and newly created `BrowserWindow` instances. Do not rely on an `app`-level `browser-window-closed` event.
- The avatar overlay window should close itself when it is the only remaining `BrowserWindow`.
- Preserve `CODEX_DESKTOP_DISABLE_LINUX_AVATAR_OVERLAY_AUTO_CLOSE=1` as a local escape hatch.

## Chrome Extension Host

Linux repacks must install a Chrome native messaging host for the Codex Chrome extension.

- Keep `installLinuxChromeExtensionHost` in `src/repack.js` wired into `installChannelRuntime`.
- The manifest path must be `~/.config/google-chrome/NativeMessagingHosts/com.openai.codexextension.json`.
- The manifest must allow `chrome-extension://hehggadaopoacecdllhhajmbjkdcmajg/`.
- The generated host must bridge Chrome native messaging stdio to `/tmp/codex-browser-use/*.sock` with the same 4-byte length-prefixed JSON framing used by Browser Use native pipes.
- The host must answer extension `ping` requests itself so the extension can show `Connected` even before a Codex task attaches.
- Keep tests for `installLinuxChromeExtensionHost` and `buildLinuxChromeExtensionHostModule` in `test/repack.test.js`.
- If the Chrome plugin fails to install or connect after an upstream update, verify both host paths: the root resource host used by the native messaging manifest (`app/resources/chrome-extension-host`) and the bundled plugin Linux wrapper (`app/resources/plugins/openai-bundled/plugins/chrome/extension-host/linux/x64/extension-host`).
- The installer diagnostic manifest must include `chromeExtensionHost` and `chromeNativeMessagingHost`; do not treat the Chrome plugin as repaired until the manifest path and wrapper executable both exist in the installed channel resources.

## Browser Use Permissions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darwinhace/codex-linux-app](https://github.com/darwinhace/codex-linux-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->

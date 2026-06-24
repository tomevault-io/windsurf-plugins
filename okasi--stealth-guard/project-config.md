---
trigger: always_on
description: This document is the canonical architecture summary for this repository as of the current source tree.
---

# Stealth Guard Architecture (Code-Verified, Current)

This document is the canonical architecture summary for this repository as of the current source tree.

## Runtime Model

- Manifest: MV2 (`manifest.json`) with a persistent background page.
- No build step: plain JavaScript files are loaded directly by the browser.
- Execution contexts:
1. Background page (`background.js` + `lib/*`) for policy orchestration and shared state.
2. Content script (`content-scripts/injector.js`) at `document_start` in all frames.
3. Injected MAIN-world inline script (built by `injector.js`) that patches browser APIs.
4. UI pages (`popup/*`, `options/*`) that read/write config through runtime messages.

## Essential Modules

| Path | Responsibility |
| --- | --- |
| `manifest.json` | Extension entry points, permissions, script order, content script registration. |
| `background.js` | Startup orchestration, runtime message hub, UA header spoofing, WebRTC policy control, proxy lifecycle, Turnstile bypass tracking, per-site session save/switch handlers (cookies + tab storage), context menus, notifications. |
| `content-scripts/injector.js` | Session-cached config bootstrap, global/feature allowlist gating, Turnstile detection + bridge, MAIN-world injection, fingerprint alert forwarding. |
| `lib/config.js` | Config defaults/schema, deep merge with persisted data, default whitelist merging. Storage key: `stealth-guard-config`. |
| `lib/storage.js` | Promise wrapper over `chrome.storage.local` (`read/write/remove/clear`). |
| `lib/domainFilter.js` | Hostname extraction + wildcard allowlist matching (`example.com`, `*.example.com`, `webmail.*`, generic `*pattern*`) with parse/regex caches. |
| `lib/proxy.js` | Proxy profile/routing helpers, bypass normalization, PAC generation, proxy mode application (`system`, `fixed_servers`, `pac_script`). |
| `popup/popup.js` | Quick toggles, per-tab triggered-feature highlighting, current-site allowlist actions, per-site session switcher controls, debounced current-tab reload after config updates. |
| `options/options.js` | Full settings UI, autosave/save/reset/import/export, proxy profile CRUD, duplicate-save suppression for unchanged configs. |

## Initialization Sequence

1. Browser loads background scripts in manifest order:
   `lib/storage.js` -> `lib/config.js` -> `lib/domainFilter.js` -> `lib/proxy.js` -> `background.js`.
2. `background.js` immediately runs `initializeBackground()`:
   - `loadConfig()` and sets in-memory `currentConfig` + cached `DomainFilter`.
   - Applies UA header spoofing listener (`webRequest.onBeforeSendHeaders`).
   - Applies base WebRTC policy (`chrome.privacy.network.webRTCIPHandlingPolicy`), deduped by last/pending policy.
   - Applies proxy settings (`chrome.proxy.settings`).
   - Rebuilds context menus.
3. `chrome.runtime.onInstalled` and `chrome.runtime.onStartup` re-apply UA/WebRTC/proxy (install also opens options page on first install).
4. For each frame at `document_start`, `injector.js`:
   - Reads session cache (`__STEALTH_GUARD_CONFIG_CACHE__`) synchronously.
   - Falls back to embedded defaults + migration shims if cache is missing/stale.
   - Refreshes from `chrome.storage.local` asynchronously with TTL guard (`__STEALTH_GUARD_CONFIG_CACHE_REFRESH_TS__`, 3s).
   - Exits early when globally disabled, no features enabled, globally allowlisted, or on challenge domains.
   - Detects Turnstile (immediate + MutationObserver + DOMContentLoaded path) and notifies background.
   - Injects MAIN-world protection script and registers message bridges.
5. Popup/options request config via runtime messages and persist updates through `update-config`.

## State Ownership and Data Flow

- Persistent source of truth:
  - `chrome.storage.local["stealth-guard-config"]`
  - `chrome.storage.local["stealth-guard-sessions"]`
  - `chrome.storage.local["stealth-guard-active-sessions"]`
- Background in-memory state:
  - `currentConfig`, `currentDomainFilter`, `configLoaded`, `initializationPromise`
  - Turnstile bypass map: `turnstileTimestamps` (hostname -> timestamp, 3-minute TTL)
  - `triggeredFeaturesPerTab` (tabId -> hostname + `Set` of triggered features)
  - Proxy transition flags: `proxyDisabledForWhitelist`, `pendingProxyDisableTabId`
  - WebRTC dedupe state: `lastAppliedWebRTCPolicy`, `pendingWebRTCPolicy`
  - Notification throttle map: `lastNotificationTime`
- Content-script session state:
  - `sessionStorage["__STEALTH_GUARD_CONFIG_CACHE__"]` (versioned with `_version`)
  - `sessionStorage["__STEALTH_GUARD_CONFIG_CACHE_REFRESH_TS__"]`
  - `sessionStorage["__STEALTH_GUARD_TURNSTILE_TS__"]`
- UI transient state:
  - Popup: debounced reload timer for rapid toggles.
  - Options: serialized snapshot dedupe (`lastSavedConfigSerialized`, `saveInFlightSerialized`).

## Interaction Points

### Runtime Message Contract (`chrome.runtime.sendMessage`)

| Direction | Type | Payload | Response |
| --- | --- | --- | --- |
| Popup/Options -> Background | `get-config` | none | `{ config }` or `{ config: null, error }` |
| Popup/Options -> Background | `update-config` | `{ config }` | `{ success: true }` or `{ success: false, error }` |
| Popup/Options -> Background | `reset-config` | none | `{ success: true }` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okasi/stealth-guard](https://github.com/okasi/stealth-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->

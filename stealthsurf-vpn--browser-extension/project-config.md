---
trigger: always_on
description: Manages proxy connections and auth tokens. Persists state in `chrome.storage.local` (Chrome MV3 kills workers after ~30s idle).
---

# AGENTS.md

Guide for AI assistants working with the StealthSurf Browser Extension codebase.

## Project Overview

Cross-browser extension (Chrome MV3, Firefox MV2) for connecting to proxies via StealthSurf VPN. Displays user's configs, paid options, and cloud server proxies. Supports one-click proxy connection (SOCKS5 on Firefox, HTTP on Chrome), location changes, split tunneling, and auto-update checking.

## Tech Stack

- React 18.3.1, Vite 6.4.1, VK UI 6.7.4, Recoil 0.7.7
- Axios with token refresh via background service worker
- @vkontakte/icons, notistack
- Biome (formatting/linting), @vitejs/plugin-react-swc, Sass

## Conventions

- Formatting: `npm run format` (Biome). Tabs + double quotes; classic JSX runtime
- Comments in code: English only. UI strings: Russian. No TypeScript
- Imports: relative paths (extensions don't support absolute `/src/...`)
- Naming: components PascalCase, utilities camelCase, hooks `use*` prefix

## Project Structure

```text
extension/
├── manifest/
│   ├── manifest.chrome.json         # Chrome Manifest V3
│   ├── manifest.firefox.json        # Firefox Manifest V2 (AMO)
│   └── manifest.firefox.github.json # Firefox Manifest V2 (GitHub, with update_url)
├── src/
│   ├── background/
│   │   ├── index.js                 # Entry: ensureInit + message listener
│   │   ├── proxyManager.js          # Cross-browser proxy abstraction
│   │   ├── proxyChrome.js           # PAC script + onAuthRequired + split tunneling
│   │   ├── proxyFirefox.js          # proxy.onRequest with inline auth + split tunneling
│   │   ├── authManager.js           # Token CRUD + PKCE code exchange + refresh via chrome.alarms
│   │   └── messageHandler.js        # Message routing with sender.id validation
│   ├── popup/
│   │   ├── main.jsx                 # Entry: RecoilRoot + App
│   │   ├── App.jsx                  # VK UI providers, auth guard, page router
│   │   ├── pages/
│   │   │   ├── AuthPage.jsx         # PKCE OAuth login
│   │   │   ├── MainPage.jsx         # Power toggle, IP badge, config selector, update banner
│   │   │   ├── ConfigSelectPage.jsx # Config list with connect buttons and ping
│   │   │   ├── LocationSelectPage.jsx # Location picker with ping measurement
│   │   │   ├── SettingsPage.jsx     # Profile, proxy settings, useful links
│   │   │   ├── FeedbackPage.jsx     # User feedback/idea submission form
│   │   │   └── SplitTunnelPage.jsx  # Split tunneling: domains + IPv4/IPv6/CIDR (exclude/include)
│   │   ├── hooks/
│   │   │   ├── useExtAuth.js        # Auth check via storage listener
│   │   │   ├── useProxyList.js      # Fetch + normalize configs from all sources
│   │   │   ├── useProxyConnection.js # connect/disconnect/restore via background
│   │   │   ├── useLoadResources.js  # Parallel data loading with retry
│   │   │   ├── useSplitTunnelSync.js # Account sync state machine (pull-on-mount, debounced push)
│   │   │   └── useSnackbarHandler.js # Snackbar helper
│   │   ├── components/
│   │   │   ├── ErrorBoundary.jsx    # Error boundary with key-based remount
│   │   │   └── MainPageSkeleton.jsx # Loading skeleton for popup initial load
│   │   └── state/
│   │       ├── atoms.js             # extensionAtom, proxyAtom, resourcesAtom, pingsAtom
│   │       └── selectors.js         # Read/write selectors
│   ├── callback/
│   │   ├── callback.html            # OAuth callback page — Firefox code capture
│   │   └── callback.js              # OAuth callback script — extracts code, sends to background
│   ├── api/
│   │   ├── api.instance.js          # Axios with chrome.storage.local tokens
│   │   └── routes/
│   │       ├── route.configs.js     # getConfigs, getSubconfig, createSubconfig, changeLocation
│   │       ├── route.paid-options.js # getPaidOptions, subconfigs, settings
│   │       ├── route.cloud-servers.js # getCloudServers
│   │       ├── route.proxies.js     # getCloudServerProxies, createCloudServerProxy
│   │       ├── route.locations.js   # getLocations
│   │       ├── route.profile.js     # getProfile
│   │       ├── route.profile-extension.js # split-tunnel account sync (GET/PUT)
│   │       └── route.feedback.js    # sendFeedback (POST /feedback)
│   ├── shared/
│   │   ├── constants.js             # MSG types, STORAGE_KEYS, CACHE_TTL_MS
│   │   ├── parseConnectionUrl.js    # protocol://user:pass@host:port → object
│   │   ├── getCurrentTimestamp.js   # Unix timestamp in seconds
│   │   ├── ping.js                  # measureBest via XHR to location ping_ip
│   │   ├── getPingLabel.jsx         # Colored ping label component (JSX)
│   │   ├── localizeDate.js          # Unix timestamp → Russian locale date string
│   │   ├── countryFlag.jsx          # Country code → CDN flag <img> (__CDN_DOMAIN__), 🌐 fallback
│   │   ├── ipUtils.js               # IPv4/IPv6/CIDR parsing + matching (popup + Firefox)
│   │   ├── pacIpHelpers.js          # ES5 PAC-safe IP helpers (inlined into PAC via ?raw)
│   │   ├── pkce.js                  # PKCE code_verifier/code_challenge generation
│   │   └── updateChecker.js         # GitHub Releases version checker (non-store)
│   └── assets/
│       ├── popup.html               # Popup HTML shell

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StealthSurf-VPN/browser-extension](https://github.com/StealthSurf-VPN/browser-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->

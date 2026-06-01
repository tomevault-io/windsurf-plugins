---
trigger: always_on
description: **Generated:** 2026-05-05
---

# NOCTURNE-UI — CAR THING WEB FRONTEND

**Generated:** 2026-05-05
**Commit:** 643cfe2
**Branch:** main
**Related repos** (separate sibling checkouts, NOT subdirs of this one): `nocturned` (the daemon this UI talks to over WS :5000), `nocturne-image` (Buildroot firmware that bakes this UI's `dist/` into the kiosk).

## OVERVIEW

Vite + React 19 SPA served by Chromium kiosk on the Spotify Car Thing (800×480, rotary dial + touch + hardware preset buttons). Talks to the `nocturned` daemon over WebSocket on port 5000 (same origin). No Spotify Web API calls from the browser — everything is proxied via the daemon.

## STACK

- **Vite 6** + `@vitejs/plugin-react-swc` (SWC, not Babel)
- **`@vitejs/plugin-legacy`** — builds for Chrome 69 target (Car Thing Chromium is old)
- React 19.2 + react-router-dom 7 + MobX 6 (mockingbird only)
- Tailwind CSS 3 + SCSS modules (mockingbird only) + `@headlessui/react`
- `bun` as package manager (`bun.lockb`, NOT `package-lock.json`)

## STRUCTURE

```
nocturne-ui/
├── index.html            # Single root div, loads src/main.jsx
├── vite.config.js        # Legacy plugin + dev-mode chrome69 transform shim
├── postcss.config.js     # Custom postcss-inset-fix plugin (chrome69 lacks `inset`)
├── tailwind.config.js    # 12-language font-family stack (Inter + Noto variants), resolved from system-installed fonts
├── eslint.config.js      # Flat config, JS only, no TS
├── .prettierrc           # EMPTY file — defaults only
├── public/fonts/         # 18 woff2 files — NOT loaded by the app. Kept so devs can install the same fonts locally to test the UI; production fonts come from the kiosk Linux system.
└── src/
    ├── main.jsx          # 6-line entry: ReactDOM.createRoot(...).render(<App />)
    ├── App.jsx           # 1420-line God component: auth flow, routing, providers
    ├── index.css         # Tailwind directives + global styles + `:root` block defining `--font-*` CSS vars (resolve to system-installed font families)
    ├── pages/Home.jsx    # Sidebar-driven home (+ sections under pages/home/)
    ├── components/       # UI components — see src/components/AGENTS.md
    ├── hooks/            # 18 hooks, ~10K lines, singleton state — see src/hooks/AGENTS.md
    ├── mockingbird/      # Alt UI (stock Spotify skin) — see src/mockingbird/AGENTS.md
    ├── contexts/         # SettingsContext, OTAContext, NotificationContext, VoiceContext
    └── utils/            # colorExtractor (album art → gradient), helpers
```

## APP FLOW

```
main.jsx
 └─ <App />
     └─ SettingsProvider → OTAProvider → NotificationProvider → VoiceProvider → DeviceSwitcherContext
         └─ <Router>  (BrowserRouter, see "Routing" below)
             └─ <UIShell isMockingbird={settings.mockingbirdUiEnabled}>
                 ├─ mockingbird true  → React.lazy(MockingbirdShell)     (mockingbird/)
                 └─ mockingbird false → {content}  (switch on activeSection/viewingContent):
                     ├─ "nowPlaying"     → NowPlaying
                     ├─ "lock"           → LockView
                     ├─ viewingContent   → ContentView (album/playlist/artist/show)
                     ├─ auth/network/... → AuthScreen / NetworkScreen / SplashScreen / Tutorial
                     └─ default          → Home (sections: recents, library, artists, radio, podcasts)
```

Overlays render outside the switch: `PairingScreen`/`MockingbirdPairingOverlay`, `NetworkBanner`, `DeviceSwitcherModal`, `ButtonMappingOverlay`, `PowerMenuOverlay`, `VoiceOverlay`, `NotificationsContainer`.

## ROUTING

**`BrowserRouter` is wrapped but no `<Route>` is declared anywhere.** It exists solely so descendants can call `useNavigate()`/`useNavigate` hooks from `react-router-dom`. Screen selection is an internal state machine driven by `App.jsx` props (`activeSection`, `viewingContent`, screen-visibility booleans) — don't add `<Route path=...>` expecting it to do anything.

## WHERE TO LOOK

| Task                                  | Location                                                                                                                                                                                                                                                                                  |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Mount order / provider stack          | `src/App.jsx` bottom (last 100 lines)                                                                                                                                                                                                                                                     |
| Which screen renders when             | `src/App.jsx` content switch (~line 1226) + `mockingbirdSystemScreen`                                                                                                                                                                                                                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usenocturne/nocturne-ui](https://github.com/usenocturne/nocturne-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

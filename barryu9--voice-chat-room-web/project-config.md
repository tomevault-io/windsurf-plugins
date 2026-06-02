---
trigger: always_on
description: - **Frontend**: React 18 + TypeScript + Vite + TailwindCSS + Zustand + mediasoup-client + Socket.io-client
---

# Project: voice-chat-room

## Stack
- **Frontend**: React 18 + TypeScript + Vite + TailwindCSS + Zustand + mediasoup-client + Socket.io-client
- **Backend**: Node.js + Express + Socket.io + Mediasoup 3.x + Mongoose (+ mongodb-memory-server for dev)
- **Deploy**: Docker + Docker Compose (mongo + backend + nginx)

## Quick Dev
```bash
# Terminal 1 - backend (nodemon auto-restart)
cd backend && npm run dev

# Terminal 2 - frontend (Vite HMR)
cd frontend && npm run dev
```
- Backend: `http://localhost:3001` (health: `/health`)
- Frontend: `https://localhost:5173` or `https://<LAN-IP>:5173` (LAN)

## Production URLs
- Frontend: `https://<your-frontend-domain>`
- Backend API/WS: `https://<your-backend-domain>`
- Server IP: `<your-server-ip>`
- RTP ports: UDP 40000-49999

## Deploy
```bash
git pull && bash deploy.sh
```
- Backend: `node:22-slim` (NOT alpine — mediasoup incompatible with musl)
- Backend binds `127.0.0.1:3001`, nginx binds `127.0.0.1:8080`
- User's own reverse proxy in front of both

## Key Design Decisions
- **No registration**: deviceId via FingerprintJS (fallback to localStorage random ID)
- **Voice-only room grid**: users appear ONLY after clicking "加入语音" (creating a producer)
- **Speaker indicator**: remote speakers use server-side fixed threshold (-55dB via AudioLevelObserver); self uses local noise gate threshold (user-adjustable, default -45dB)
- **Remote audio playback**: native `<audio>` elements (NOT Web Audio API) — more reliable across browsers
- **Voice disconnect**: emits `producer:close` via Socket.io to reliably trigger `USER_LEFT` broadcast
- **DB**: MongoDB for persistent config (channels, bans, settings); Node.js Map for volatile WebRTC state
- **Dev MongoDB**: mongodb-memory-server auto-starts when no MONGODB_URI set

## Architecture Flow
```
login → room:join (no USER_JOINED yet)
  → click "加入语音" → create Producer
    → broadcast USER_JOINED + NEW_PRODUCER
    → other clients consume and play via <audio>
  → click "断开语音" → emit producer:close
    → broadcast USER_LEFT (if last producer)
    → user removed from grid
```

## Common Issues Fixed
- dotenv path: `__dirname` is in `src/config/` → need `../..` to reach backend root
- `store.producer` was storing Transport, not Producer → fixed to capture `transport.produce()` return value
- Stop order: close transport BEFORE producer, otherwise `transportclose` doesn't fire
- AudioContext resume must be awaited
- Windows npm: must use `cmd /c "npm ..."` in PowerShell due to execution policy
- `npx tsc --noEmit` for TS check, `npx vite build` for production build

## Rules for Admin Panel Settings
- **All settings must show current values as defaults** — never show empty/placeholder values when editing
- **Every save action must give user feedback** — use `showToast('xxx已更新', 'success')` after backend ACK
- Settings must be re-fetched from server after save to ensure consistency (`admin:config-getall`)
- Use `useEffect` to sync UI state with current values when the panel opens

---

# Design Specification

## 1. UI Conventions

### 1.1 Colors
| Usage | Class |
|-------|-------|
| Primary accent | `primary-500` (#6366f1), `primary-600` (#4f46e5) |
| Success/green | `green-400`, `green-500` |
| Danger/red | `red-400`, `red-500` |
| Warning/yellow | `yellow-400`, `yellow-500` |
| Panel background | `glass-panel` (semi-transparent + backdrop-blur) |
| Card background | `glass-card` |
| Input background | `bg-gray-800/60` |
| Input border | `border border-gray-600/50` |
| Text primary | `text-white` |
| Text secondary | `text-gray-400` |
| Text muted | `text-gray-500` |
| Placeholder | `placeholder-gray-600` |

### 1.1.1 Theme Design Philosophy
- The appearance system is split into two independent dimensions: **appearance style** and **primary color**.
- Appearance styles currently include `暗夜`, `日光`, `纯粹`, and `深夜`. `暗夜` and `深夜` are dark-mode appearances; `日光` and `纯粹` are light-mode appearances. The default is `暗夜 + 紫色`.
- Primary colors currently include common presets such as `红`, `黄`, `蓝`, `绿`, `青`, `紫`, `粉`, and `橙`, plus `自定义`. Custom colors are stored separately for dark and light modes, because the same hex value may not read well in both modes.
- `暗夜` uses the dark glassmorphism foundation: dark body background, translucent panels/cards, white primary text, light-gray secondary/muted text, and accent-colored glow.
- `日光` keeps the glassmorphism structure but uses warm white or very pale tinted backgrounds with soft accent glows. Primary text must be black or near-black, and secondary/muted text must be dark gray; do not use overly pale gray text in light appearances.
- `纯粹` is a flatter light special case: pure white background, minimal gradients, and doodle-like line borders. It still follows the selected primary color, but the treatment should feel hand-drawn rather than glossy.
- `深夜` is a flatter dark special case: pure black background, minimal gradients, and primary-color glowing borders.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [barryu9/Voice_Chat_Room_Web](https://github.com/barryu9/Voice_Chat_Room_Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->

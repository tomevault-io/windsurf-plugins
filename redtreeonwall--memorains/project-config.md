---
trigger: always_on
description: npm install                  # also copies Excalidraw fonts to public/
---

# Memorains Note — Agent Instructions

## Build & Run Commands

### Client (React + Vite)

```bash
cd client
npm install                  # also copies Excalidraw fonts to public/
npm run dev                  # Vite dev server, http://localhost:5173
npm run build                # tsc + vite build + sw-version.js
npm run lint                 # ESLint check
npm run lint:fix             # ESLint auto-fix
```

### Server (Express + WS, CommonJS target)

```bash
cd server
npm install
npm run build                # tsc → build/
npm run server               # node build/index.js (requires running DB)
npm run dev                  # podman-compose up -d + tsc --watch + node (IS_DEV=true)
```

### Desktop (Electron)

```bash
cd client
npm run desktop:dev          # tsc electron/ + electron . --no-sandbox
npm run desktop:package      # electron-forge package
npm run desktop:make         # electron-forge make (all platforms)
npm run desktop:make:win     # cross-compile for Windows
```

### Mobile (Capacitor — Android)

```bash
cd client
npm run mobile:build         # VITE_BUILD_CAPACITOR=true vite build
npm run mobile:open:android  # build + sync + open in Android Studio
npm run mobile:sign:android  # signed release APK → built-apk/memorains-release.apk
```

The signing keystore lives at `client/memorains.keystore`. `client/scripts/build-android-release.sh` copies it into the Capacitor-managed `android/` dir before building.

### Production Package

```bash
cd script
bash build_package.sh        # → package.tar.gz (client/dist + server + DB schema + nginx + docker-compose)
```

### Deploy

```bash
tar -zxvf package.tar.gz
cd package
mkdir -p ~/certificate       # place cert.pem + cert.key
podman compose up -d
# browse: https://<host>/doc/client/
```

### Build Scripts (`script/`)

| Script | Purpose |
|--------|---------|
| `build_web_package.sh` | Production deploy tarball (`out/package.tar.gz`: client/dist + server/ + DB + nginx + docker-compose) |
| `build_client_package.sh` | Desktop (.deb + .zip) and Android .apk → `out/` |
| `build_all.sh` | Runs both of the above in sequence |
| `sync_interface.sh` | Copies `HttpMessage.ts` + `DataEntity.ts` from `server/src/interface/` to `client/src/interface/` (does **not** sync `UserServerMessage.ts` — that file must be synced manually) |

**Note:** All scripts must be run from the `script/` directory (they use `` script_dir=`pwd` `` rather than `$(dirname "$0")`).

---

## Architecture

### Server: Multi-Process Model

The server spawns **N child processes** (one per CPU core × 2, or 2 in dev), each running `DocServerImp.ts` on a different WebSocket port (8081+). The **main process** hosts:

- `DocApplicationImp` — entry point, initializes `DocServerManagerImp` and `UserServerImp`
- `DocServerManagerImp` — manages child processes via `child_process.fork()`, routes document-open requests to the least-loaded child, cleans up zero-user rooms
- `UserServerImp` — Express HTTP server on port 8000 (auth, CRUD, document room token issuance)
- `DataBaseManagerImp` — MariaDB pool (host: `reno_note_mariadb` in prod, `127.0.0.1` in dev)

**Key flow for opening a document:**

1. Client → `POST /doc/server/docRoomInfo` (with JWT)
2. `UserServerImp` checks permissions, calls `DocServerManagerImp.requestOpenDoc(docId)`
3. `DocServerManagerImp` picks the least-loaded child process, sends `M2C_OpenMessageRequest` via IPC
4. Child process loads the document state from MariaDB into a Yjs `Y.Doc`, returns room password + port
5. Server issues a short-lived JWT (`roomToken`) scoped to docId+userId
6. Client opens WebSocket to `wss://host/doc/websocket/<docId>/<userId>/<password>/<roomToken>`

### Client: Editor Abstraction

Four editor types share a common wrapper:

- **QuillEditor** — rich text (URL path `/document`)
- **ExcalidrawCanvas** — infinite canvas (`/canvas`)
- **TodoListEditor** — task manager (`/todo`)
- **ChatEditor** — messenger-style chat (`/chat`)

Every editor is wrapped by `CommonEditor`, which provides:
- Document title bar, save/sync status indicators, offline/reconnect banners
- User presence indicators (colored circles per collaborator)
- Sidebar (`SideList`) navigation between documents
- Keyboard shortcut management (`ShortcutManager`)

**Core data flow in the client:**

```
NoteDocument          ← Yjs Doc (source of truth)
    ↓
MessageBridge         ← WebSocket (sends/receives Yjs updates + cursors)
    ↓
CommonEditor          ← UI wrapper (title, status, reconnect)
    ↓
QuillEditor / ExcalidrawCanvas / TodoListEditor / ChatEditor  ← binding layer
```

- `NoteDocument` owns the `Y.Doc` and coordinates local persistence (IndexedDB) with remote sync
- `MessageBridge` owns the WebSocket lifecycle, handles reconnection with exponential backoff (max 5 retries), and relays `ServerMessage` to listeners
- Editor-specific components bind a Yjs type (e.g., `ydoc.getText("quill")`) to the UI via framework-specific adapters (`y-quill` for Quill)

### Offline & Sync Strategy

- All documents are stored in IndexedDB (`client/src/DB/IndexedDB.ts`) — the `document` object store mirrors `DocumentEntity`
- On open: local data loads first for instant render; then the WebSocket syncs diffs via Yjs state vectors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redTreeOnWall/memorains](https://github.com/redTreeOnWall/memorains) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

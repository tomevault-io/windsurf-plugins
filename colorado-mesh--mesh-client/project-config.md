---
trigger: always_on
description: This file is self-contained. ARCHITECTURE.md and CONTRIBUTING.md are human references — read them only if you need deep subsystem detail beyond what's here.
---

# AGENTS.md — Coding Guidelines for AI Assistants

This file is self-contained. ARCHITECTURE.md and CONTRIBUTING.md are human references — read them only if you need deep subsystem detail beyond what's here.

## 1. Scope & Workflow

- Only change what was asked. No drive-by refactors, reformatting, or types/comments outside scope.
- **Testing:** Ship a passing test for behavioral changes; do not call the task done without it.
- **Stateful/I/O code:** Preserve integrity on failure; document failure point, fallback, and logging where it matters.

## 2. Architecture & Domain

Electron: `src/main/` (Node, SQLite, BLE, MQTT), `src/preload/` (bridge), `src/renderer/` (React 19, Vite, Zustand). **Dual-protocol:** meshtastic and meshcore; gate UI with `ProtocolCapabilities` and `useRadioProvider(protocol)` (do not compare `protocol === 'meshcore'`). Routing/diagnostics changes must stay compatible with the Diagnostics panel (Hop Goblins, Hidden Terminals, etc.). **pnpm** only for package commands. **Never** add cryptocurrency tech or dependencies.

**Colors:** Use Tailwind CSS utility classes (e.g., `text-green-400`, `bg-slate-700`). Custom theme colors via CSS custom properties in `styles.css` (`--color-brand-green`, etc.). Avoid inline hex colors in JSX.

**Code style and testing:** [Code style & standards](CONTRIBUTING.md#code-style--standards) and [Testing protocols](CONTRIBUTING.md#testing-protocols) in [CONTRIBUTING.md](CONTRIBUTING.md).

### Layout map

Path alias `@/*` → `src/*` (see `tsconfig.json`).

| Boundary | Path            | Role                                                                                                                                                                                                                  |
| -------- | --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Main     | `src/main/`     | SQLite (`database.ts`, `db-compat.ts`), BLE (`noble-ble-manager.ts`), MQTT (`mqtt-manager.ts`, `meshcore-mqtt-adapter.ts`), logging (`log-service.ts`, `sanitize-log-message.ts`), IPC handlers, window, GPS, updater |
| Preload  | `src/preload/`  | `contextBridge` exposing namespaced `electronAPI` only; never expose `ipcRenderer`                                                                                                                                    |
| Renderer | `src/renderer/` | React 19 + Vite + Zustand: `components/`, `hooks/`, `stores/`, `lib/` (includes `lib/diagnostics/`, `lib/radio/`, `lib/transport/`), `workers/`                                                                       |
| Shared   | `src/shared/`   | IPC contracts (`electron-api.types.ts`), protocol-neutral helpers                                                                                                                                                     |

Entry points: `src/main/index.ts`, `src/preload/index.ts`, `src/renderer/main.tsx`, `src/renderer/App.tsx`.

### Dual protocol

Both stacks can run simultaneously. Feature-gate with `ProtocolCapabilities`:

```typescript
import { useRadioProvider } from '@/lib/radio/providerFactory';
const capabilities = useRadioProvider(protocol);
```

### IPC data flow

Adding a cross-boundary feature:

1. Types in `src/shared/electron-api.types.ts`.
2. `ipcMain.handle('namespace:action', ...)` in `src/main/index.ts`.
3. Expose on `electronAPI` in `src/preload/index.ts` via `ipcRenderer.invoke`.
4. Call from renderer: `window.electronAPI...`

## 3. Security & Error Handling

- Catches must log, rethrow, or `// catch-no-log-ok <reason>`. Prefer Result types over deep nesting.
- **Logging:** `console.debug` / `warn` / `error` as appropriate; no bare `console.log`.
- **Log injection:** Call `sanitizeLogMessage()` on user-controlled strings before `appendLine()` or loggers.
- **IPC:** Namespaced channels (`db:*`, `mqtt:*`, etc.); expose only via `contextBridge` in preload; **never** expose `ipcRenderer` directly.
- **System boundaries:** Follow repo security rules for subprocess APIs, DOM/HTML sinks, and dynamic code. Validate external inputs; do not over-validate internal code.

## 4. Code Style

- **Prettier:** Semi always, single quotes, trailing commas, print width 100, tab 2, LF.
- **TypeScript:** Strict; avoid `any`; prefer `unknown` + guards; export types; prefer interfaces over type aliases.
- **React:** Function components only; `exhaustive-deps` is errors; `?.` in JSX; every interactive control needs `aria-label`.
- **Zustand:** Module-level defaults for stable refs; prefer `useStore(s => s.field)` over broad subscriptions; avoid subscribing to whole Maps when one id suffices; `persist` for localStorage, IPC from an effect for SQLite; extract time constants to `src/renderer/lib/timeConstants.ts`.
- **Performance:** No hot-path O(n); lazy cleanup when collections grow large.

## 5. Testing

- Renderer: jsdom (`src/renderer/**/*.test.{ts,tsx}`). Main: node (`src/main/**/*.test.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Colorado-Mesh/mesh-client](https://github.com/Colorado-Mesh/mesh-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->

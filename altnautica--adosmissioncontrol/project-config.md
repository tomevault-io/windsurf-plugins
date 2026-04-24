---
trigger: always_on
description: **See [CLAUDE.md](../CLAUDE.md) for full architecture, checklist patterns, and detailed examples.**
---

# ADOS Mission Control — Copilot Instructions

**See [CLAUDE.md](../CLAUDE.md) for full architecture, checklist patterns, and detailed examples.**

---

## Core Architecture Rules (Non-negotiable)

### 1. Protocol Abstraction — Always Use `DroneProtocol`
- **Never** call MAVLink or MSP functions directly from components.
- All FC operations go through `DroneProtocol` interface (`src/lib/protocol/types.ts`).
- Access protocol: `useDroneManager((s) => s.getSelectedProtocol)`.
- Example: Parameter writes → `protocol.setParameter('PARAM_NAME', value)`, never raw MAVLink.

### 2. State Management — Zustand Only
- **Never** use `useState` for shared data. All shared state lives in stores (`src/stores/`).
- 34 specialized stores: telemetry, mission, geofence, rally, patterns, agent, etc.
- Telemetry uses `RingBuffer<T>` (fixed-size circular arrays, no unbounded growth).
- Subscribe with selectors: `useStore((s) => s.field)`, not `useStore()` (re-renders all).

### 3. FC Panels — Use `usePanelParams` Hook
- Every panel that reads/writes FC parameters must use `usePanelParams` (`src/hooks/use-panel-params.ts`).
- Separate `paramNames` (required) and `optionalParams` arrays for firmware compatibility.
- Always use `PanelHeader` for loading/error/refresh UI.
- Wrap writes in `useArmedLock()` (blocks when armed) and `useUnsavedGuard()` (warns on nav).

### 4. MAVLink Lifecycle
- **ArduPilot auto-saves to EEPROM** on `PARAM_SET` — do not block on the flash commit ACK.
- `commitParamsToFlash()` is fire-and-forget (sends `MAV_CMD_PREFLIGHT_STORAGE` as belt-and-suspenders).
- CRC_EXTRA must match MAVLink XML — wrong CRC silently fails frame validation.

---

## Code Style & TypeScript

### TypeScript Strict Mode
- **No `any` types.** Use `unknown` and narrow explicitly.
- **Explicit return types** on all public APIs and exported functions.
- `interface` for object shapes; `type` for unions and type aliases.

### Styling (Tailwind v4 — Dark-First)
- Background: `bg-surface-primary` (near-black).
- Text: `text-text-primary` (white).
- Accent: `text-accent-primary` (electric blue).
- **Never** hardcoded colors, `bg-white`, or `text-black`.
- **Never** use arbitrary values (`w-[347px]`) if a design token exists.

### Client Directives
- All interactive components need `"use client"` (Next.js App Router).
- FC panels, indicators, and store consumers are always client components.

---

## Convex Backend — Critical Gotchas

### Setup
- `.env.local` must have `NEXT_PUBLIC_CONVEX_URL` (created by `npx convex dev`).
- Run `npm run convex:dev` in a separate terminal, keep it running during development.
- `convex/_generated/` is committed and must be regenerated after schema changes (`npx convex dev`).

### Using Queries/Mutations
- **Always guard with `useConvexAvailable()`:**
  ```tsx
  const convexAvailable = useConvexAvailable();
  const data = useQuery(api.myFunction, convexAvailable ? {} : "skip");
  ```
- Missing skip guard → `"Convex client not found"` error when `NEXT_PUBLIC_CONVEX_URL` is undefined.
- Demo mode (`npm run demo`) also disables Convex — guard all queries.
- Reference: [use-convex-skip-query.ts](../src/hooks/use-convex-skip-query.ts).

### Cloud Mode
- Auto-activates on HTTPS (`window.location.protocol === "https:"`).
- On HTTP (local dev), uses direct agent connection.

---

## Quick Reference: Implementation Patterns

### Add a New FC Panel
1. Create `src/components/fc/YourPanel.tsx` with `"use client"`.
2. Define `paramNames` array (required) and `optionalParams` array.
3. Call `usePanelParams({ paramNames, optionalParams, panelId: "your-panel", autoLoad: false })`.
4. Render `<PanelHeader>` with loading/error states.
5. Add mock params to `src/mock/mock-params.ts` (type: 9 for all).
6. Test in demo mode: `npm run demo` → Read from FC → Edit → Save → Flash.
7. **Full checklist**: See [CLAUDE.md § Checklist: New FC Panel](../CLAUDE.md#checklist-new-fc-panel).

### Add a MAVLink Message Decoder
1. Add Message ID + CRC_EXTRA to lookup table in `src/lib/protocol/mavlink-parser.ts`.
2. Create decoder case in parser's `decodePayload()` switch.
3. Define callback type in `src/lib/protocol/types.ts`.
4. Add callback array + emit in `src/lib/protocol/mavlink-adapter.ts`.
5. Add stub in `src/mock/mock-protocol.ts`.
6. (Optional) Add encoder in `src/lib/protocol/mavlink-encoder.ts`.
7. **Full checklist**: See [CLAUDE.md § Checklist: New MAVLink Message Decoder](../CLAUDE.md#checklist-new-mavlink-message-decoder).

### Add an Indicator Component
1. Create `src/components/indicators/YourIndicator.tsx` with `"use client"`.
2. Subscribe to store with selector; use `useTelemetryFreshness()` for stale detection.
3. Export from `src/components/indicators/index.ts`.
4. **Full checklist**: See [CLAUDE.md § Checklist: New Indicator Component](../CLAUDE.md#checklist-new-indicator-component).

### Add a Zustand Store
1. Create `src/stores/my-store.ts`.
2. Use `RingBuffer<T>` for telemetry (fixed capacity, e.g., 1000 entries).
3. Wire into `bridgeTelemetry()` in `src/stores/drone-manager.ts`.
4. Use selectors in consumers: `useMyStore((s) => s.specificField)`.
5. **Full checklist**: See [CLAUDE.md § Checklist: New Zustand Store](../CLAUDE.md#checklist-new-zustand-store).

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [altnautica/ADOSMissionControl](https://github.com/altnautica/ADOSMissionControl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->

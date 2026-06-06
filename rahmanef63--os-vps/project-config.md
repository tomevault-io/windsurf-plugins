---
trigger: always_on
description: Mobile-first web cockpit for a headless VPS, from any browser. Desktop-style UI
---

# os-vps (product name: **Topside**)

Mobile-first web cockpit for a headless VPS, from any browser. Desktop-style UI
metaphor over a vertical-slice stack; value is utility (terminal/files/monitor/
browser), not an OS. Repo/service/domain keep the `os-vps` slug; "Topside" is the
UI brand. **Self-contained**: a single Next.js app, no database, no external
agent — it runs AS a host process and controls its own machine.

- Stack: Next 16 (App Router) · React 19 · Tailwind 4 · shadcn/ui · TypeScript.
  **No `middleware.ts` — `proxy.ts`** (Next 16 rename).
- Auth: password + device approval → HMAC signed-cookie session (`lib/auth/`).
  No Convex, no Clerk.
- Host access: `lib/host/` does fs/exec/sys directly (Node `fs` + `child_process`),
  bounded by `OS_FS_READ_ROOTS` / `OS_FS_WRITE_ROOTS`.
- Layout: `app/` + `frontend/slices/<slug>/`; barrel-only cross-slice imports
  (`@/features/<slug>`).

## Read first
- `README.md` — what it is, features, security model, quickstart.
- `.env.example` — every env var.
- `docs/` — `ARCHITECTURE.md` + `PLAN.md` are current (self-contained). `PROGRESS.md`
  is the running log (newest on top; Phase 17 = AppShell framework, Phase 18 = routing);
  `DESIGN-RECONCILE.md` is stamped archive (old Convex/agent design);
  `MOBILE-RESPONSIVE-PLAN.md` = the deferred "Phase E" sweep (primitives ARE built in
  `appshell/primitives/`, but NOT yet adopted across app UIs — that changes the frontend).

## Architecture
```
browser ──https──> os-vps (Next.js :4005) ──┬── lib/host → Node fs/child_process (host)
              signed-cookie auth (lib/auth)  └── os-browser (Playwright, :4002, optional)
```
- `/api/v1/*` = the os-rr Cloud API (fs/exec/sys/browser), every route `verifyAuth`
  (session cookie) first. Client picks mock (default) vs live in Settings → Server.
- `/api/auth/*` = login/logout/me/devices. `/api/config` = BYOK AI key.
- Persistence is local: window layout + app registry in localStorage; device
  allowlist + config in `~/.os-vps/*.json`.

## AppShell framework (the shell is generic + rr-liftable)
The shell is NOT one slice. It is split so the whole desktop+mobile shell can lift
to `resources/` (rr) and drive any project from one manifest:
- `frontend/slices/appshell/` — the **generic, brand-free** framework: window runtime
  + desktop/mobile surfaces, app/feature/brand registries, `<Slot region>`,
  `ResponsiveProvider`/`useResponsive` + the 4 DRY primitives, the pub/sub buses
  (toast/activity/inspector), and `<AppShell manifest>` (the one entry point). It
  imports NO brand/feature and NO os-vps `@/lib/*` — only the universal `@/lib/utils`
  (`cn`). Everything project-specific arrives via `manifest.capabilities`.
- `shell-search` / `shell-inspector` / `shell-notifications` / `shell-control-center`
  / `shell-widgets` — each shell **feature** is its own slice, mounts into a named
  `<Slot>` via `defineFeature({ id, slots, provider? })`, and is also consumer-free
  (data via capabilities, not `@/lib`). Buses live in core so apps fire them without
  depending on a feature slice.
- `os-shell` — the thin os-vps **consumer**: `shell.manifest.ts` (Topside brand + app
  list + slugs + features) + `capabilities.ts` (adapts `@/lib/appearance`+`os-api`+
  `ai/stream` to `ShellCapabilities`) + a re-export barrel (`@/features/os-shell`
  re-exports appshell verbatim, so all app slices stay unedited).
- **Windowing** (`appshell/lib/store.ts`): `openWindow(app,title,size,payload,{multi})`.
  Default = single instance per app (reuse/focus); `AppDescriptor.multi` (e.g. Files)
  spawns a fresh window each open. `focusApp(id)` reveals the front-most existing
  window without spawning — used by `UrlSync` so deep-links/back-forward don't
  duplicate a multi app. **Window coords (`win.x/y`) are relative to the desktop
  `<section top-[30px]>`, NOT the viewport** — snap/maximize geometry must use
  `workArea()` (section-relative: `top=GAP`, `bottom=vh-TOPBAR-DOCK_RESERVE`), the
  drag snap preview must be `position:absolute` (shares the surface), and drag
  commits must use `offsetLeft/offsetTop`, never viewport `getBoundingClientRect`.
- **`window-content.tsx` loads app bundles with `useState`/`useEffect`, NOT
  `React.lazy`+`Suspense`.** Window opens come from the synchronous external store
  (`useSyncExternalStore`); a Suspense boundary suspending in that path misses its
  retry ping — the chunk resolves but the spinner only clears on the next render
  (a click). A `setState` on import-resolve always re-renders. Don't reintroduce
  Suspense here. Dock hover warms the chunk (`app.load()`), so it stays instant.
- **Dock = macOS behaviour**: clicking a running app focuses its front window
  (`focusApp`, never spawns); hovering shows its open windows to switch + a "New
  Window" entry for `multi` apps. Opening surfaces (Launchpad/Spotlight) spawn.
- **`ShellCapabilities`** is the injection seam: `useAppearance`, `useCpuPercent`,
  `useSearch`→`SearchHit[]`, `useSystemStats`, `useChat`, `useServerToggle`. Defaults
  merged in `CapabilitiesProvider` so optional caps degrade (accessors stay
  unconditional). Add an app = manifest edit; add a shell feature = new `shell-*` slice
  + `defineFeature` + list in `TOPSIDE_FEATURES`. No surface edits (open/closed).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rahmanef63/os-vps](https://github.com/rahmanef63/os-vps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-06 -->

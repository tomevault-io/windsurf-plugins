---
trigger: always_on
description: Mobile-first web cockpit for a headless VPS, from any browser. Desktop-style UI
---

# mso (product name: **MSO**)

Mobile-first web cockpit for a headless VPS, from any browser. Desktop-style UI
metaphor over a vertical-slice stack; value is utility (terminal/files/monitor/
browser), not an OS. Repo/service/domain keep the `mso` slug; "MSO" is the
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
- `docs/PROGRESS.md` — **THE source of truth for what exists.** Newest entry at top;
  read the top few and you know the current state. Everything else in `docs/` is
  history or a plan, and history goes stale.
- `README.md` — what it is, features, security model, quickstart.
- `.env.example` — every var you can actually set. Reconciled against `process.env`
  in code on 2026-08-03 (Camoufox, memory/threads paths, `NEXT_DEPLOYMENT_ID`,
  `NEXT_PUBLIC_COMMIT_SHA` were all missing and were added). What is still deliberately
  absent is what you never set by hand: framework vars (`NEXT_RUNTIME`,
  `NEXT_PUBLIC_BUILD_ID` — injected by `next.config`), systemd's (`NOTIFY_SOCKET`,
  `WATCHDOG_USEC`), the OS's (`PATH`, `SHELL`), test-only ones (`E2E_BASE_URL`,
  `OPENCLAW_HOME`), and `OS_BROWSER_*`, which belong to the retired `os-browser/`
  sidecar and not to this app. Still grep `process.env` before adding a new one.
- **The code wins over any doc.** `docs/ARCHITECTURE.md` is HISTORY, not current — it
  still describes a retired Playwright browser sidecar, a managed-app "single-origin
  mode" that was removed for security, and workspace modes that were reversed. It is
  kept for the reasoning it records, not as a description of today.
- `frontend/slices/assistant/CONTRACT.md` — current, and authoritative for what an
  Agent / Tool / Skill / Playbook is and what reaches the model.

**The doc set, and the one rule.** `docs/PROGRESS.md` is the SSOT; everything else is
history, a live plan, or reference. Append to PROGRESS when you ship — do not start a
second log. (`docs/CHANGELOG.md` was exactly that and was merged back in; the root
`progress.md` is gitignored local scratch and claims authority it does not have.)
Deleted 2026-07-28 as dead: `SHELL-INTEGRATION-PLAN.md` and `SYNC-PLAN.md` (both target
sibling repos that do not exist on this machine), `browser-agent-plan.md` (the retired
Playwright sidecar), `SIXFIX-PLAN.md` (a finished dated fix list). Nothing linked to any
of them. Deleted 2026-07-30 in the same spirit: `PLAN.md` (the "master plan" — every
section contradicted by shipped code, and its one unique asset, a Control-Room-vs-MSO
table, is descriptive rather than decision-carrying) and `MULTISHELL-PLAN.md` (its sibling
repo is gone, all six phases are checked off, and PROGRESS.md:574 reverses its one unique
decision). Both recoverable with `git show bccd0b1:docs/<name>`.

## Architecture
```
browser ──https──> mso (Next.js :4005) ──── lib/host → Node fs/child_process (host)
              signed-cookie auth (lib/auth)
```
The Browser app is **Camoufox** — a real anti-fingerprinting Firefox on a headless
X display on this host, streamed in over noVNC through `/camoufox-vnc/*` (gated in
`proxy.ts` by the same verified-session check that guards `/api/v1/exec`). It
replaced BOTH the old Playwright sidecar (`os-browser`, :4002, retired) and the
sandboxed-iframe browser that briefly followed it — an iframe cannot render most of
the web, because X-Frame-Options refuses framing on the majority of real sites.
`os-browser/` stays in-repo only as dev tooling (scripts/e2e use its Playwright
install). See the Browser/camoufox note further down for the systemd user unit.
- `/api/v1/*` = the host API (fs/exec/sys/term/apps/camoufox/managed-apps), every
  route `verifyAuth` (session cookie) first. There is no `/api/v1/browser`.
  Client picks mock (default) vs live in Settings → Server.
- `/api/auth/*` = login/logout/me/devices. `/api/config` = BYOK AI key.
- Persistence is local: window layout + app registry in localStorage; device
  allowlist + config in `~/.mso/*.json`.

## AppShell framework (the shell is generic + rr-liftable)
The shell is NOT one slice. It is split so the whole desktop+mobile shell can lift
to `resources/` (rr) and drive any project from one manifest:
- `frontend/slices/appshell/` — the **generic, brand-free** framework: window runtime
  + desktop/mobile surfaces, app/feature/brand registries, `<Slot region>`,
  `ResponsiveProvider`/`useResponsive` + the 4 DRY primitives, the pub/sub buses
  (toast/activity/inspector), and `<AppShell manifest>` (the one entry point). It
  imports NO brand/feature and NO mso `@/lib/*` — only the universal `@/lib/utils`
  (`cn`). Everything project-specific arrives via `manifest.capabilities`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rahmanef63/mso](https://github.com/rahmanef63/mso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->

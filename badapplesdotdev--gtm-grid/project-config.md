---
trigger: always_on
description: Operational notes for AI agents (and humans) working in this repo.
---

# AGENTS.md

Operational notes for AI agents (and humans) working in this repo.

## Verification flow

- **Every new user-facing desktop feature ships an Electron E2E test.** Add a
  Playwright spec under `packages/desktop/e2e/` that drives the real built app.
  Unit/component coverage is welcome, but it does not replace the E2E.
- **Run Electron E2E locally in the background before opening a PR.** Use
  `pnpm e2e:background`; it detaches the long run and writes its output under the
  ignored `.gtmgrid/e2e/` directory, so the active agent turn and user are not
  blocked by Playwright output. Continue useful work, then use
  `pnpm e2e:status`. Only inspect `pnpm e2e:log` when progress or a failure needs
  diagnosis. The launcher refuses to overlap two runs in one worktree.
- `listing-shots.spec.ts` is an asset-generation task, not verification. It is
  excluded from normal E2E because it overwrites tracked marketing images; run
  it explicitly with `pnpm --filter @gtmgrid/desktop e2e:listing-shots`.
- **Electron E2E is deliberately local-only, not a GitHub Actions gate.** CI
  keeps the fast `pnpm lint`, `pnpm typecheck`, `pnpm test`, and web-build gates;
  releases repeat the code checks. A PR containing a user-facing desktop feature
  is not ready until its detached local E2E run reports `passed`.

## Running the desktop app against **staging** (Electron → staging backend)

Use this for OAuth work. Staging has its **own database** (Supabase branch) and its
**own OAuth apps**, so you can connect, disconnect and break things without
touching production data or production grants.

| | staging | production |
|---|---|---|
| Backend | `https://staging.gtmgrid.dev` | `https://www.gtmgrid.dev` |
| Git branch | `staging` | `main` |
| Database | Supabase branch `pkbxzbnkpwjawifnlrct` | `fmzqedfoqhdzpdsguvci` |
| Vercel env | custom environment `staging` | Production |

### Build the PACKAGED app (what you want for OAuth)

`electron:pack` — not the un-packaged run below — because the OAuth callback
bounces the browser back through a `gtmgrid://` deep link, and that protocol only
registers with the OS for a **packaged** app. Un-packaged, consent completes but
the hand-back silently does nothing and the connect card just keeps polling.

```bash
cd packages/desktop
VITE_API_URL="https://staging.gtmgrid.dev" \
VITE_INNGEST_URL="https://staging.gtmgrid.dev" \
VITE_PARTY_URL="" \
VITE_POSTHOG_KEY="" \
CSC_IDENTITY_AUTO_DISCOVERY=false \
pnpm electron:pack           # → packages/desktop/release/mac-arm64/GTM Grid.app
```

`VITE_PARTY_URL` is deliberately EMPTY: staging has no PartyKit deployment, and
the renderer treats it as optional (`|| undefined`), so realtime multiplayer is
simply inactive. Everything else — auth, grids, columns, OAuth — works.
`CSC_IDENTITY_AUTO_DISCOVERY=false` skips code signing, which you do not have
certs for locally and do not need for a local test build.

**The endpoints are baked in at BUILD time.** There is no runtime switch: a
staging app and a production app are different binaries. If the app is talking to
the wrong backend, you rebuilt with the wrong `VITE_API_URL` — check
DevTools → Network, not the app's settings.

### ⚠️ `electron:pack` BREAKS the shared node_modules — repair it afterwards

Not a theoretical risk. Verified: after `electron:pack`, `pnpm test` fails with

```
.pnpm/better-sqlite3@11.10.0/.../better_sqlite3.node
NODE_MODULE_VERSION 130   ← Electron 33's ABI
requires NODE_MODULE_VERSION 127   ← Node 22
```

`electron:bundle` runs `electron-rebuild -f -w better-sqlite3 -m sidecar`. The
`-m sidecar` flag LOOKS like it confines the rebuild to
`packages/desktop/sidecar/` (which has its own `npm install`), and that reasoning
is wrong: electron-rebuild follows the symlink chain back into the pnpm store at
`~/repos/gtm-grid/node_modules/.pnpm/` and rebuilds THAT copy for Electron's ABI.

That store is shared by every worktree via a `node_modules` symlink, so one
`electron:pack` breaks `pnpm test` and `pnpm server` in this worktree AND in every
sibling worktree, until repaired.

**Repair (required after every `electron:pack`):**

```bash
D=~/repos/gtm-grid/node_modules/.pnpm/better-sqlite3@11.10.0/node_modules/better-sqlite3
rm -rf "$D/build"                       # the Electron-ABI binary
( cd "$D" && npx prebuild-install --runtime=node --target=$(node -p "process.versions.node") )
pnpm test                               # confirm green again
```

`pnpm rebuild better-sqlite3` does **not** fix it — `prebuild-install` no-ops when
`build/` already exists, so it reports "Done" while leaving the Electron binary in
place. The `rm -rf build` is the load-bearing step.

Only the packaged build does this. `pnpm build` (renderer only) and
`pnpm electron:dev` are safe.

### Signing in

Staging's database is EMPTY — no accounts carry over from production. Sign up
fresh, then create a workspace before opening Tools.

Email OTP works. Google OAuth needs `AUTH_GOOGLE_*` on the staging environment;
if it is unset, use email OTP.

### Testing the Slack connection

1. **Tools → Slack.** The Connect button must be **enabled**. If it reads
   "Slack isn't set up on this deployment yet", `SLACK_CLIENT_ID` is missing from

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [badapplesdotdev/gtm-grid](https://github.com/badapplesdotdev/gtm-grid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->

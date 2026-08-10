---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

`vercel-openclaw` is a single-instance Next.js 16 app that manages exactly one persistent Vercel Sandbox running OpenClaw: auth, on-demand create/resume, proxy at `/gateway`, HTML injection for WebSocket rewrite and gateway-token handoff, firewall learning, and channel webhooks (Slack, Telegram, Discord, WhatsApp).

For operator docs, start with `docs/getting-started/README.md`. It is the main handoff for the three-repo system (`vclaw`, `vercel-openclaw`, and the OpenClaw fork), `vclaw create`, operational paths, release, and reliability contracts. Then use `README.md`, `CONTRIBUTING.md`, and the deep docs under `docs/` (`architecture.md`, `channels-and-webhooks.md`, `lifecycle-and-restore.md`, `preflight-and-launch-verification.md`, `deployment-protection.md`, `environment-variables.md`, `api-reference.md`).

## Commands

Package manager is `pnpm`. Tests use `node:test`.

```bash
pnpm install
pnpm dev
node scripts/verify.mjs             # canonical CI entrypoint; runs lint/test/typecheck/build
node scripts/verify.mjs --steps=test,typecheck
pnpm check:verify-contract          # guards documented env vars across README/CLAUDE.md/CONTRIBUTING.md/.env.example
pnpm smoke:remote --base-url https://my-app.vercel.app [--destructive] [--auth-cookie "session=..."]
```

Use `node scripts/verify.mjs` for all automation and CI — never bare `npm`/`tsx`. For docs-only changes that touch operator env names or instructions, also run `pnpm check:verify-contract`.

## Main guide / getting started

Read `docs/getting-started/README.md` before changing setup, provisioning, release, bundle compatibility, or cross-repo behavior. The guide was integrated from `~/dev/vclaw-handoff` and is now the canonical in-repo onboarding path.

Guide pages:

- `docs/getting-started/system-map.md` — three-repo architecture and ownership boundaries.
- `docs/getting-started/operational-paths.md` — create/deploy, sandbox boot/proxy, channel delivery, and verification boundaries.
- `docs/getting-started/vclaw-create.md` — supported install flow, flags, non-interactive runs, and failure boundaries.
- `docs/getting-started/release-and-reliability.md` — OpenClaw bundle assets, dashboard verification, CLI publish, and risk areas.

Keep `README.md`, `docs/README.md`, and this file in sync when the guide moves or its scope changes. `AGENTS.md` is a symlink to this file, so updates here are also the agent-facing instructions.

## Local dev against prod env

For production debugging, prefer the linked directory produced by `vclaw create --auto-link --dir ~/dev/vercel-openclaw`. Passing `--dir` is intentional: it deploys and links this local checkout instead of the default managed clone under `~/.vclaw/<scope>/<project>/app`. Use the managed clone for clean install testing, but use `--dir ~/dev/vercel-openclaw` whenever validating local fixes, bundle-compatibility changes, admin/debug patches, or anything not yet present in the managed clone source. `--auto-link` writes `.vercel/project.json`, pulls the admin secret, automation bypass secret, and vclaw project metadata into `.env.local`, and updates `.gitignore` so the local secret file stays untracked. Debug from that directory so admin scripts, Vercel project targeting, and the source checkout all agree.

To tweak the admin UI locally while reading real production data:

1. Start from the `--auto-link` directory, or run `vclaw create --auto-link --dir ~/dev/vercel-openclaw` against the target project.
2. Edit `.env.local`:
   - set `VERCEL_ENV=development` so `isVercelDeployment()` flips and Redis connects (`src/server/store/store.ts:57`)
   - set `LOCAL_READ_ONLY=1` so every admin mutation returns `403 { error: "LOCAL_READ_ONLY" }` (`src/server/auth/route-auth.ts`)
   - unset `VERCEL_AUTH_MODE` so admin-secret mode works locally
3. `pnpm dev`, then `POST /api/auth/login` with `ADMIN_SECRET` for the session cookie

`.env.local` contains live admin and deployment-protection secrets. Do not commit it, paste it into logs, or include it in `.agent-runs` artifacts. `getSandboxController()` returns the real v2 SDK whenever `NODE_ENV !== "test"` (`src/server/sandbox/controller.ts:201-211`). Without `LOCAL_READ_ONLY`, `POST /api/admin/stop` from localhost stops the prod sandbox.

## Architecture map

| Subsystem | Entry points |
| --------- | ------------ |
| Store (metadata + side keys) | `src/server/store/{store,redis-store,memory-store,keyspace}.ts`, `src/shared/types.ts` |
| Sandbox lifecycle | `src/server/sandbox/{lifecycle,controller}.ts` |
| OpenClaw bootstrap + restore | `src/server/openclaw/{config,bootstrap}.ts`, `src/server/openclaw/restore-assets.ts` |
| Proxy + HTML injection | `src/app/gateway/[[...path]]/route.ts`, `src/server/proxy/{proxy-route-utils,htmlInjection,waitingPage}.ts` |
| Firewall | `src/server/firewall/{domains,policy,state}.ts` |
| Channels | `src/server/channels/{driver,state,webhook-urls,connectability}.ts`, `src/server/channels/{slack,telegram,whatsapp,discord}/` |
| Auth | `src/server/auth/{admin-auth,admin-secret,session,vercel-auth,route-auth}.ts` |
| Public URL resolution | `src/server/public-url.ts` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/vercel-openclaw-archived](https://github.com/vercel-labs/vercel-openclaw-archived) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->

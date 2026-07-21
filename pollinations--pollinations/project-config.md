---
trigger: always_on
description: Two-phase review via `apps-review-submissions.yml` (AI + human). Source of truth: `apps/APPS.md`.
---

# Agent Guidelines for pollinations.ai

## App Submission Handling

Two-phase review via `apps-review-submissions.yml` (AI + human). Source of truth: `apps/APPS.md`.

Flow: user opens issue with `TIER-APP` → workflow validates + AI generates preview → bot posts `APP_REVIEW_DATA` JSON + labels `TIER-APP-REVIEW` → maintainer adds `TIER-APP-APPROVED` → workflow prepends row to `apps/APPS.md`, opens PR with auto-merge, closes issue via `Fixes #NNN`.

Label state machine:
- `TIER-APP` → `TIER-APP-REJECTED` (duplicate/invalid) | `TIER-APP-INCOMPLETE` (not registered) | `TIER-APP-REVIEW` → `TIER-APP-APPROVED` (merged) | `TIER-APP-REJECTED` (closed)

Manual edits: edit `apps/APPS.md`, run `node .github/scripts/app-update-greenhouse.js`.

APPS.md columns: `Emoji | Name | Web_URL | Description (~80 chars) | Language (ISO code, no flags) | Category | Platform | GitHub (@user) | GitHub_ID | Repo | Stars (⭐N) | Discord | Other | Submitted_Date (issue created) | Issue_URL (#N) | Approved_Date (PR merged)`.

Platforms (auto-detected; comma-separated for multi): `web` (default w/ URL), `android`, `ios` (App Store or routinehub.co), `windows`, `macos`, `desktop` (cross-platform), `cli`, `discord`, `telegram`, `whatsapp`, `library` (npm/PyPI/SDK), `browser-ext`, `roblox`, `wordpress`, `api` (default w/o URL).

Categories: `image`, `video_audio`, `writing`, `chat`, `games`, `learn`, `bots`, `build`, `business`.

## Discord

Guild ID `885844321461485618` (https://discord.gg/pollinations-ai-885844321461485618) — use for Discord MCP tools.

## Repository Structure

- `enter.pollinations.ai/` — Auth gateway + billing (Cloudflare Worker)
- `gen.pollinations.ai/` — Edge router + text generation Worker
- `image.pollinations.ai/` — Image GPU/backend assets; public gateway code lives in `gen.pollinations.ai/`
- `pollinations.ai/` — React frontend
- `packages/sdk/` — `@pollinations/sdk` (client + React hooks)
- `packages/mcp/` — `@pollinations/mcp` (MCP server; see `packages/mcp/AGENTS.md`)
- `shared/` — auth, registry, IP queue; `shared/registry/` holds model registries
- `apps/` — Community apps + `APPS.md`
- `social/` — Discord/Reddit/GitHub automation

## API Gateway

Primary: `https://gen.pollinations.ai` → routes to `enter.pollinations.ai` for auth/billing.

- Auth: `pk_` (frontend), `sk_` (backend). Keys: https://enter.pollinations.ai/keys
- Billing: Pollen credits ($1 ≈ 1 Pollen). Full docs: `./APIDOCS.md`
- Pack checkout: Stripe. Polar is retired from runtime; do not add Polar SDKs,
  Worker bindings, webhooks, or automated writes. Historical Polar handling
  (pre-Stripe pack revenue, Nov 2025–Jan 2026) lives in the economics ingest
  connector prompt (`apps/operation/economics/ingest/agent.system.txt`).
- Services: Text (Portkey, multi-provider), Image (gen Worker dispatch to providers/GPU backends), Video (Wan/Veo/LTX), Audio (ElevenLabs, TTM)
- Wallet: Pollen is earned by completing Quests; balances live in the `tier_balance` (shown as Quest Pollen) and `pack_balance` (Paid) buckets. The legacy `tier` D1 column and `tier_balance` wire name are kept for compatibility; see `shared/db/better-auth.ts`.
- Referral links must use the canonical landing page with a short `?ref=` value; record analytics behind the page instead of exposing a tracking API as the destination URL.

### Local Development

Ports: enter `3000` (API at `/api/*`), gen `8788`. Run `npm run dev` per service.

Image generation now runs inside `gen.pollinations.ai`; local image API tests should target the gen worker on port `8788`.

Local API test:
```bash
curl "http://localhost:8788/image/test?model=flux" -H "Authorization: Bearer $TOKEN"
curl "http://localhost:8788/v1/chat/completions" -H "Authorization: Bearer $TOKEN" ...
```

## API Quick Reference

- Image: `GET gen.pollinations.ai/image/{prompt}` (bearer token)
- Text (OpenAI): `POST gen.pollinations.ai/v1/chat/completions` with `{model, messages}` (bearer token)
- Simple text: `GET gen.pollinations.ai/text/{prompt}?key=...`
- Audio: `GET gen.pollinations.ai/audio/{text}?voice=nova&key=...`
- Models: `/image/models`, `/v1/models`
- See `./APIDOCS.md`, `.claude/skills/enter-services/SKILL.md`

## ⚠️ YAGNI — You Aren't Gonna Need It (CRITICAL)

**Follow YAGNI religiously:**

- Only implement what's needed now. Remove unused functions.
- No speculative abstractions, "just in case" helpers, preemptive test utils/wrappers.
- No backward-compat fallbacks — clean breaks beat bloat. When changing tokens/headers/APIs, update all consumers at once.
- When user says "keep it simple" — one function, one price, one config. Simplest thing that works.

## Cloudflare Production Deployment Safety

**CRITICAL — production Cloudflare deployments must always run through GitHub Actions:**

- Use the service's production deployment workflow, such as `Deploy / gen.pollinations.ai`; use `workflow_dispatch` when path filters do not trigger it.
- Never run `wrangler deploy --env production`, a production deployment npm script, or a direct production Worker upload from a local machine or agent session.
- If CI credentials lack a required permission, update the scoped GitHub Actions secret and rerun the workflow. Never bypass CI with a local Cloudflare OAuth session.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pollinations/pollinations](https://github.com/pollinations/pollinations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

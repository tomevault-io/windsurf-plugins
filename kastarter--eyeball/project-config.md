---
trigger: always_on
description: Open-core tool and integration platform for AI agents: one typed, authenticated API across SaaS, messaging, voice, social data, and business systems.
---

# eyeball

Open-core tool and integration platform for AI agents: one typed, authenticated API across SaaS, messaging, voice, social data, and business systems.

## Stack

- TypeScript strict mode, Node.js 24+, pnpm 11, Turborepo, Hono, Vitest, and Biome.
- Dashboard: Next.js 16, React 19, Tailwind CSS 4, and semantic CSS tokens.
- Docs renderer: Next.js 16, React 19, Tailwind CSS 4, `next-mdx-remote`, Shiki, and `remark-gfm`.
- Core schema validation: Ajv Draft 2020-12 plus `ajv-formats`.

## Conventions

- Public package exports use ESM `.js` specifiers from `src/index.ts` barrels.
- Release and docs copy must distinguish checked-in private Cloud source from a deployed hosted service and must not imply npm or live-provider certification.
- Changesets keeps `core`, `catalog`, `toolkits`, and `sdk` in one fixed version group; every Node app and the experimental bridge are explicitly ignored and remain private.
- Canonical tools use `toolkit.operation`; restricted names use reversible `toolkit__operation`.
- `/v1/*` is API-key/project scoped; `/health` and `/ready` are public. `/health` is dependency-free liveness; `/ready` is no-store, fail-closed traffic admission.
- Staged-file uploads use padded-base64 JSON; defaults are 25 MiB and one hour via `EYEBALL_FILE_MAX_BYTES` / `EYEBALL_FILE_TTL_MS`, with a pre-buffer body ceiling for encoded content plus 16 KiB metadata overhead.
- Credential env vars use `EYEBALL_CRED_<TOOLKIT>_*`; `EYEBALL_API_KEYS` accepts `key:project[:user]`.
- Manifest `endpoint.baseUrlOverrideEnv` values are the only trusted provider endpoint override seam.
- HTTP and provider tests prefer Hono `app.request`; do not require loopback sockets.
- Real Cloud integration tests load the ignored sibling checkout conditionally so OSS-only clones keep passing.
- Webhooks sign `<unix-seconds>.<raw-body>` as `v1=<HMAC-SHA256 hex>`; attempts time out at 10s and retry after 0s/30s/2m/10m/1h.
- Webhook URL validation strips trailing DNS root dots before rejecting localhost, `.localhost`, `.local`, and literal private-network targets; delivery never follows redirects.
- Executor logs and telemetry attributes pass through central redaction; never emit credentials, authorization headers, canonical bodies, webhook secrets, or file bytes.
- OpenTelemetry exporters are disabled unless `EYEBALL_OTEL=1`; tests use in-memory providers and never require a collector.
- Trigger events deliver as `trigger.<toolkit>.<name>` through signed webhooks; push ingest secrets appear only in create-time URLs.
- Unauthenticated push-trigger ingest is capped at 1 MiB before buffering; exposed path credentials rotate through the subscription rotation endpoint.
- `EYEBALL_DATABASE_URL` enables the executor's five-connection Postgres pool, including durable redacted trigger-event history, staged-file metadata/content, and voice-agent resources, and applies committed executor Drizzle migrations at boot; absent keeps all zero-config in-memory defaults.
- Executor HTTP limits share project buckets: standard 120/min with 240 burst, execute 60/min with 120 burst; `EYEBALL_RATE_LIMIT_*` overrides them and daily quota is off by default.
- Cloud execution usage reserves after project throttling and before record allocation; execution stores preflight idempotent replays, and terminal reports reuse one opaque SHA-256 identity.
- Usage reservations release on failures before adapter dispatch; attempted adapter calls report through the terminal outbox.
- The docs shell follows Mintlify-derived geometry: a 56px top bar, 576px prose column, and 256px/264px navigation rails.
- Dashboard cloud mode is explicit: `NEXT_PUBLIC_EYEBALL_MODE=cloud` selects cloud-backed features and server-only `EYEBALL_CLOUD_URL` supplies the control-plane origin; unset remains demo mode.
- Dashboard and landing Vercel projects use app-local `vercel.json` files with `apps/dashboard` and `apps/landing` as their Root Directories; their filtered builds run from the monorepo root so workspace dependencies and the root pnpm lockfile remain authoritative.
- Dashboard cloud requests use the same-origin `/api/cloud` allowlist proxy; org/project context and manually pasted per-project executor keys live in validated `HttpOnly` cookies.
- Cloud Stripe returns land on session-gated `/billing/checkout/success` and `/billing/checkout/cancel`; `/billing?org=...` renders organization billing, current-month usage, plan comparison, checkout, and portal controls.
- Cloud API-key verification is authenticated `POST /internal/keys/verify` with a pre-buffer 4 KiB body cap and a 1,024-character key schema; never place customer keys in internal URL queries.
- Hosted executor auth checks static `EYEBALL_API_KEYS` before Cloud verification; remote caches use SHA-256 key digests with 60s positive/5s negative defaults, and `EYEBALL_CREDENTIALS=cloud` uses the executor-owned HTTP credential client.
- Authenticated dashboard, provider, webhook, and remote-voice HTTP clients use manual redirect handling; remote voice requires HTTPS outside explicit loopback and supplied control tokens are at least 32 characters.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kastarter/eyeball](https://github.com/Kastarter/eyeball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

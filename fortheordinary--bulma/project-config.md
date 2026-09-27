---
trigger: always_on
description: Rules for any AI agent working in the Bulma repo. Read before writing code or docs.
---

# AGENTS.md

Rules for any AI agent working in the Bulma repo. Read before writing code or docs.

## 0. Agent instruction file

`AGENTS.md` is the single source of truth for agent rules in this repo. Ignore other conventional files (`CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`, etc.) even if your tool auto-loads them. `CLAUDE.md` is a symlink to this file; do not edit it directly. If you find guidance only in one of those other files, treat it as stale and surface it for migration into `AGENTS.md`.

## 1. Terminology — scoped prohibition

Bulma is **"Agentic global account for remote workers"**. It is built on BlindPay's stablecoin rails, so internal code, schema, comments, and `.plans/` docs may use the literal blindpay vocabulary (USDC, USDT, wallet, Polygon, network, signature, etc.) — pretending otherwise would make the code unreadable against blindpay docs.

**Customer-facing surface area is strict fiat.** Anywhere the user sees output, crypto vocabulary is banned:

- CLI command output (`bulma balance` → "Balance: $1,234.56 USD", never "1234.56 USDC")
- API response field names and human-readable messages
- Error messages shown to users
- Marketing / README / public docs
- Commit messages and PR descriptions affecting public surface

Forbidden in customer-facing copy: crypto, cryptocurrency, blockchain, web3, stablecoin, USDC, USDT, wallet (use "account"), on-chain, chain names (Polygon, Ethereum, etc.), gas, mint, burn, bridge.

Internal surface (code, schema columns, log lines, `.plans/`, AGENTS.md, internal types) — use the technically correct term; do not coin euphemisms that diverge from BlindPay's API.

When in doubt about which side of the line a string lives on, ask.

## 2. Tech stack (authoritative)

| Layer            | Choice                                                                                |
| ---------------- | ------------------------------------------------------------------------------------- |
| Runtime / PM     | Bun (`>=1.2`)                                                                          |
| Monorepo         | Turborepo                                                                              |
| Language         | TypeScript (strict), Zod-first (schemas drive types)                                   |
| API              | Hono on Cloudflare Workers + `@hono/zod-openapi`                                       |
| Auth             | better-auth (Drizzle adapter, Google social plugin, Bearer plugin)                     |
| Database         | Cloudflare D1 (SQLite) via Drizzle ORM + drizzle-kit                                   |
| Secrets (dev)    | `.env` at each `apps/*` root (wrangler 4+ auto-loads for `wrangler dev`; vite for www). Optional: Infisical project; `bun run dev` pulls fresh secrets from the `/api` folder into `.env.infisical` per start. Secrets are namespaced per app (`/api`, `/www`) — see §5 Infisical secret layout. |
| Public ingress   | Cloudflare Tunnel (`cloudflared`) — `local.bul.ma` → `localhost:8787` for BlindPay webhooks. `bun run --filter=api tunnel` to start. |
| Web              | Vue 3.6 + Vapor mode + Tailwind v4 + shadcn-vue (reka-ui), Vite                        |
| Lint / Format    | oxlint + oxfmt                                                                         |
| Apps             | `apps/api` (Hono Worker), `apps/cli` (Bun CLI), `apps/www` (Vue SPA)                   |
| Shared packages  | `packages/typescript` (tsconfig), `packages/oxc` (lint cfg)                            |

**Do not introduce** other runtimes (Node-specific APIs), other ORMs (Prisma, Kysely), other validators (Yup, Valibot), other auth libs (Lucia, Clerk, Auth.js/NextAuth, Supabase Auth), other UI frameworks (React, Svelte, Solid), other CSS systems (CSS-in-JS, Sass), other linters/formatters (ESLint, Prettier, Biome), or other package managers (npm, pnpm, yarn). If a real need arises, propose it to the user before installing.

### Commit identity — agent@bul.ma only
Every commit (author **and** committer) MUST be `agent@bul.ma`. Enforced at three layers:
- `.githooks/pre-commit` — blocks a commit when `git config user.email` ≠ `agent@bul.ma`.
- `.githooks/pre-push` — blocks pushing any commit whose author/committer ≠ `agent@bul.ma`.
- `.github/workflows/verify-author.yml` — CI gate on every PR + push to `main` (catches `--no-verify` / web edits).

Hooks live in `.githooks` (version-controlled) but `core.hooksPath` is local config, so **after cloning** run once:
```sh
git config user.name "agent-bulma" && git config user.email agent@bul.ma
git config core.hooksPath .githooks
```

### 2a. Zod-first

Define a Zod schema, then derive the type with `z.infer<typeof Schema>`. Do **not** hand-write `interface` or `type` declarations for data that crosses any boundary (API I/O, DB rows, CLI args, config files, env vars). Exceptions: utility types, generics, branded primitives, internal helpers with no runtime shape.

### 2b. Hono routes use `@hono/zod-openapi`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fortheordinary/bulma](https://github.com/fortheordinary/bulma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

---
trigger: always_on
description: Cross-tool guide for any AI coding agent working in this repo (Claude Code, Cursor, Codex, Copilot, Windsurf, …).
---

# AGENTS.md — the primer for coding agents

Cross-tool guide for any AI coding agent working in this repo (Claude Code, Cursor, Codex, Copilot, Windsurf, …).
Codex, Cursor, and Copilot read a repo-root `AGENTS.md` by convention — **this file is the source of truth.** Claude Code also reads [`CLAUDE.md`](./CLAUDE.md); Cursor reads [`.cursor/rules.md`](./.cursor/rules.md). Both are short mirrors that point back here.

Read this **before writing code**. It tells you where everything lives so you don't reinvent what already exists.

---

## 1. The mental model — the buckets

This is a **bun-workspace monorepo** wrapped by **Nx** (task graph + enforced boundaries + generators). Every package has a role defined by _one question: is it served, and to whom?_

**Three buckets are what you RUN; a fourth (`packages/`) is what you SHIP.**

| Folder       | Role                                | Served?                          | Examples                                                                                                                      |
| ------------ | ----------------------------------- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `apps/`      | what **humans** see                 | public UI                        | `@stack/web` (Next.js), `@stack/landing` (marketing), `@stack/mobile` (Expo/React Native)                                     |
| `services/`  | what has a **URL** / its own deploy | served to other code             | `@stack/api` (Hono + OpenAPI), `@stack/payment` (Creem adapter), `@stack/ai-worker` (background, no URL)                      |
| `libs/`      | **shared** code                     | **never served** — consumed only | `@stack/ui`, `@stack/auth`, `@stack/db`, `@stack/ai`, `@stack/analytics`, `@stack/email`, `@stack/config`, `@stack/api-types` |
| `packages/`  | what you **ship** — a distributable | served to **third parties**      | `@stack/widget` (embeddable widget: IIFE `<script src>` + ESM); npm SDKs and CLIs live here too                              |

The first three are **what you RUN** — sorted by _who they're served to_ (your humans, your machines, your own code). `packages/` is the odd one out: **what you SHIP** — a built artifact exposed to people _outside_ your system (published to npm, embedded on a customer's site). Its tag is `type:package`; it may depend on `libs/*` only, and it's **terminal — nothing internal imports a package** (§3, law 9). If you ship nothing external, delete the folder.

If you're about to create a file, first decide which bucket it belongs to. If it doesn't obviously fit one, ask — don't invent a _fifth_ top-level folder.

## 2. The map — all 17 packages

```
builders-stack/
├── apps/
│   ├── web/          @stack/web       Next.js App Router — renders @stack/ui, live Better Auth login
│   ├── landing/      @stack/landing   public marketing site (@stack/ui hero + shared <Analytics/>)
│   ├── blog/         @stack/blog      static MDX blog — the GEO showcase, passes check:seo
│   └── mobile/       @stack/mobile    real Expo / React Native starter rendering shared @stack/ui tokens
├── services/
│   ├── api/          @stack/api       Hono + OpenAPI (/health, /docs) — validates @stack/api-types, mounts Better Auth
│   ├── payment/      @stack/payment   Creem adapter + Mock provider + webhooks (/checkout)
│   └── ai-worker/    @stack/ai-worker background load, NO URL (queue worker)
├── libs/
│   ├── ui/           @stack/ui        shadcn components + tokens (web + RN) + Storybook
│   ├── auth/         @stack/auth      Better Auth config (boot-verified end to end)
│   ├── db/           @stack/db        Drizzle schema + client (the single ORM)
│   ├── ai/           @stack/ai        provider-agnostic model client (Vercel AI SDK)
│   ├── analytics/    @stack/analytics <Analytics/> provider + isomorphic typed event catalog (./events)
│   ├── email/        @stack/email     Resend + React Email: typed, previewable templates + sendEmail()
│   ├── config/       @stack/config    typed env: one Zod schema + cached getEnv()
│   ├── api-types/    @stack/api-types the shared API contract (Zod schemas + inferred types)
│   └── seo/          @stack/seo       the one door for page metadata + JSON-LD (enforced by check:seo)
├── packages/         what you SHIP (not run) — distributables served to third parties
│   └── widget/       @stack/widget    embeddable feedback widget: IIFE (<script src>) + ESM build; type:package, libs-only, terminal
├── infra/            Dockerfiles, docker-compose, k8s (your deploy config)
├── scripts/          deploy.sh, tunnel.sh, seed.sh, link-env.sh
├── api-collection/   Bruno API collection (version-controlled requests)
├── agents/           the deep dive: skills, subagents, mcp.json (this file links there)
├── docs/             getting-started · costs · ai · architecture · nx · portless · analytics · email · secrets
├── .devops/Tiltfile  the runtime manifest — what boots and how
├── nx.json           task graph + boundary tags
└── tsconfig.base.json  shared compiler options (never fork)
```

## 3. The laws — do not break these


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lonormaly/builders-stack](https://github.com/lonormaly/builders-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->

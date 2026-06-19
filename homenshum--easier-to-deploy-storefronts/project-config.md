---
trigger: always_on
description: Build a multilingual storefront + owner inbox + passwordless auth for a solo service business — one owner at the counter, many customers passing through. Convex (state) + Vercel (functions + static) + Resend (email) + OpenRouter (translation). Use this skill any time you start a new solo-operator project (pet-sitter, tutor, photographer, freelance editor, mobile mechanic) and need the same shape: editable storefront, single inbox, magic-link owner login, no developer in the loop after deploy.
---


# easier-to-deploy-storefronts — building a one-owner / many-customers stack

You're about to build a website for a solo operator — one human running a service business who needs a presence on the web but isn't a developer. They will edit prices on their phone, see customer messages in a single inbox, and sign in via email link. You will deploy once and never log into the production database again.

This skill is the **code-construction** playbook. For the **dev process around the code** — clarifying scenarios before you write anything, designing UX with [`parity-studio`](https://github.com/HomenShum/parity-studio), enforcing per-surface changelogs with [`easier-to-read-submissions`](https://github.com/HomenShum/easier-to-read-submissions), evaluating agents, and the scaling math past the free tier — read [`DEV_FLOW.md`](DEV_FLOW.md). It's the meta-process; this is the implementation.

Ten phases, in order. Each phase explains **what** to build, **why** that design (the calls that aren't obvious), and **how** (concrete code and commands). The phases are sequenced so you can stop after any phase and have something working — Phase 4 alone gives you a multilingual marketing site; adding Phase 5 makes the owner self-sufficient.

The whole thing runs on free tiers for steady-state operation. Estimated time to a deployed v1 with a non-technical owner: **6–8 hours** for someone new to Convex, **2–3 hours** for someone who's done it before.

---

## Phase 0 — Decide whether this skill applies

This skill fits a project with **all** of:

- One owner (not a multi-tenant SaaS, not a multi-vendor marketplace).
- Many anonymous customers who don't need accounts (they fill a form and get an email reply).
- Owner is non-technical or wants to stay that way.
- Content (prices, copy, photos, FAQs) changes more than once a quarter.
- Multilingual is a real requirement, not aspirational. (English-only? Skip Phase 8.)

If the project has a **second class of authenticated user** (employees, partners, vendors), stop and use a different skill — `easier-to-deploy-storefronts` is deliberately scoped to one-owner. Adding a second admin role doubles the auth surface; it's not a small change.

---

## Phase 1 — Provider setup (30 min)

Create accounts and grab keys. Don't paste keys into chat or anywhere except the env file.

| Provider | What for | Free tier covers |
|---|---|---|
| Vercel | Static hosting + serverless functions | Unlimited Hobby projects, 100GB bandwidth/mo, 100K function invocations/day |
| Convex | DB + file storage + cron + HTTP routes | 1GB storage, 1M function calls/mo, no credit card |
| Resend | Transactional email | 3000 emails/mo, 100/day, one verified domain free |
| OpenRouter | LLM router for translation + reply assist | Pay-as-you-go; ~$0.001–$0.01 per translation. Free fallback models available |

```bash
# Scaffold the project
npm create convex@latest <your-name>      # creates /convex with schema.ts + http.ts stubs
cd <your-name>
npx convex dev                            # one-time login + creates dev deployment

# Add Vercel
npm i -g vercel
vercel link                               # creates .vercel/, links to a Vercel project

# Project structure that emerges:
.
├── convex/                # backend (schema, functions, http routes, crons)
├── api/                   # Vercel serverless functions
├── index.html             # public landing page (English)
├── zh/index.html          # Chinese
├── es/index.html          # Spanish
├── inbox.html             # owner SPA
├── .env.example           # documented env vars
├── vercel.json            # rewrites: /admin → /inbox.html, /api/* → functions
└── package.json
```

`vercel.json` should contain at minimum:

```json
{
  "rewrites": [
    { "source": "/admin", "destination": "/inbox.html" },
    { "source": "/admin/", "destination": "/inbox.html" }
  ]
}
```

The `/admin → /inbox.html` rewrite is a small UX detail that matters: the owner remembers `your-site.com/admin`, not `/inbox.html`.

---

## Phase 2 — Convex schema (45 min)

Five tables. Define them all in `convex/schema.ts` upfront — even the ones you won't use until Phase 6 — so the schema is self-documenting.

```ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  // ============== Customer-facing ==============
  inquiries: defineTable({
    name: v.string(),
    contact: v.string(),                       // email or phone
    message: v.string(),
    dog: v.optional(v.string()),               // adapt: "child" / "vehicle" / "project" for your domain
    dates: v.optional(v.string()),
    service: v.optional(v.string()),
    lang: v.optional(v.string()),
    detectedLang: v.optional(v.string()),      // OpenRouter source-language detection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HomenShum/easier-to-deploy-storefronts](https://github.com/HomenShum/easier-to-deploy-storefronts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

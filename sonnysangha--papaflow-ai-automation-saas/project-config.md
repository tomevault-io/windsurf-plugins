---
trigger: always_on
description: n8n-style workflow automation SaaS. Users (as Clerk organisations) bring their own AI keys, connect chat apps, and build workflows either on a React Flow canvas or by describing them to a Pro-only Builder agent. Every run is durable on Vercel Workflows. Working title.
---

# PapaFlow

n8n-style workflow automation SaaS. Users (as Clerk organisations) bring their own AI keys, connect chat apps, and build workflows either on a React Flow canvas or by describing them to a Pro-only Builder agent. Every run is durable on Vercel Workflows. Working title.

The full plan with research, connector matrices, code sketches and gotchas is in `docs/PLAN.md`. Read it before starting any phase. This file is the short version plus the rules.

## Stack (decided, don't relitigate)

- Next.js (App Router) on Vercel, Fluid compute
- Convex for all app state and realtime (`useQuery` subscriptions drive the canvas)
- Clerk: Organizations for workspaces, Clerk Billing (B2B) for plans, native Convex integration (session token carries `aud: "convex"`)
- Vercel Workflows / Workflow SDK (`workflow@5.0.0-beta.47`; `npm i workflow` installs 4.x — always `workflow@beta`; read only https://workflow-sdk.dev/v5/docs/ or `node_modules/workflow/docs`, the unversioned /docs/ pages are v4) for durable runs, hooks and sleep
- eve (Vercel's agent framework, beta, Node 24) for the Runtime agent (Agent node) and the Builder agent
- AI SDK 7 with direct provider packages (`@ai-sdk/openai`, `@ai-sdk/anthropic`, `@ai-sdk/google`, etc.) built per call from the org's decrypted key
- React Flow (`@xyflow/react` v12), zod, Resend for app-owned email

Pin exact versions: `next 16.3.4`, `react 19.2.8`, `convex 1.45.0`, `@clerk/nextjs 7.8.4`, `@clerk/backend 3.17.0`, `@xyflow/react 12.11.6`, `zod 4.5.4`, `ai 7.0.90` (all `@ai-sdk/*` at the versions in docs/research/versions.md), `workflow 5.0.0-beta.47`, `eve 0.49.0`. Toolchain: `typescript` 5.9.3 (never 7 — typescript-eslint peers <6.1.0), `eslint 9.39.5` (never 10 — eslint-config-next plugins peer ^9), `"engines": { "node": "24.x" }`. Never install `@clerk/themes` (Core 2 line; use `@clerk/ui`) or `@workflow/ai` (DurableAgent is deprecated; `WorkflowAgent` from `@ai-sdk/workflow`). Do not upgrade them mid-phase.

## Commands

```bash
pnpm dev              # next dev (+ eve dev server via withEve, + convex dev in another terminal)
pnpm convex:dev       # CONVEX_ALLOW_ANONYMOUS=false npx convex dev (a non-TTY shell without .env.local otherwise creates a silent local anonymous deployment)
pnpm workflow:web     # npx workflow web  (local run inspector)
pnpm typecheck        # tsc --noEmit
pnpm lint
pnpm test             # vitest
```

Build command on Vercel (in `vercel.ts`): `npx convex deploy --cmd 'pnpm build' --cmd-url-env-var-name NEXT_PUBLIC_CONVEX_URL`. `CONVEX_DEPLOY_KEY` is the deploy-time Convex var on Vercel: a production deploy key scoped to Production, a preview deploy key scoped to Preview; `NEXT_PUBLIC_CONVEX_URL` is injected by `convex deploy` into the build process only. The eve services build as separate Vercel services and never see it, so Production and Preview also carry a plain `CONVEX_URL` with the same deployment URL (see "Env vars").

## Layout

```
app/                      Next.js routes and pages
next.config.mts           withEve(withWorkflow(nextConfig), { agents: { runtime, builder } }) — .mts because eve/next is ESM-only
proxy.ts                  clerkMiddleware(); matcher excludes /_next, static files, .well-known/workflow/ and eve/
connectors/               one file per provider: how a user connects (fields, test call, discover), separate from nodes/
  (app)/w/[workflowId]/   canvas editor
  f/[workflowId]/         public form trigger page
  api/hooks/…             generic webhook trigger
  api/events/{provider}/  signed inbound events (Slack, Discord, Telegram, Stripe, GitHub…)
  api/oauth/{provider}/   authorize + callback
  api/connections/        credential save routes (seal into Convex)
convex/                   schema, queries, mutations, httpAction webhooks (Clerk)
workflows/                "use workflow" functions: run-graph.ts
workflows/steps/          "use step" functions: run-node.ts, …
nodes/                    one file per node type (see "Node definition" below), registry.ts
lib/oauth/                generic OAuth2 module + providers.ts configs
lib/vault.ts              AES-256-GCM seal/open (Node crypto)
lib/ai/providers.ts       providerFor(provider, apiKey) → AI SDK model factory
agents/runtime/           eve agent behind the Agent node (agent.ts, instructions.md, tools/, channels/eve.ts)
agents/builder/           eve Builder agent (Pro only) — same layout; tools live in agents/builder/tools/
docs/PLAN.md              the plan
```

## Node definition (the one pattern everything hangs off)

```ts
export const slackPostMessage = defineNode({
  type: "slack.postMessage",
  name: "Slack: Post message",
  category: "communication",
  credential: "slack",                 // connection kind this node needs, or null
  requiresFeature: "pro_connectors",   // Clerk feature slug or null
  inputs: z.object({ channel: z.string(), text: z.string() }),   // generates the config form + JSON Schema for the Builder
  outputs: z.object({ ts: z.string(), channel: z.string() }),    // powers the variable picker

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sonnysangha/papaflow-ai-automation-saas](https://github.com/sonnysangha/papaflow-ai-automation-saas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->

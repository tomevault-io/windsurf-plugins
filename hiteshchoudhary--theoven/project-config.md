---
trigger: always_on
description: > The batteries-included Bun framework. Express-simple, FastAPI-smart, everything configurable.
---

# Oven

> The batteries-included Bun framework. Express-simple, FastAPI-smart, everything configurable.

This file is the durable memory for this project. **Read it fully at the start of every session.**
It records what Oven is, every architectural decision we have locked (and why), and the rules
for working in this repo. `TODO.md` tracks what is done and what is next.

---

## 1. What we are building

Oven is a backend framework for **Bun**. It replaces Express and takes inspiration from
**FastAPI**, but is TypeScript-native end to end.

The core promise: **you should never wire infrastructure by hand again.**

Want auth? Add the auth brick. Want a queue? Add the queue brick. Want S3, mail, a database?
Add a brick. Oven does every piece of setup, wiring, connection management and lifecycle
handling for you, then hands you a small, obvious, fully-typed API on the request context.

```ts
// src/app.ts — bricks chain, and each one's type flows into the context (D5)
export const app = createApp()
  .use(db(drizzlePostgres({ url: env.string('DATABASE_URL'), schema })))
  .use(auth(basicAuth({ db: client, secret: env.string('AUTH_SECRET') })))
  .use(storage(s3Storage({ bucket: 'uploads' })))
  .use(mail(resendMail({ apiKey: env.string('RESEND_KEY'), from: 'hi@acme.com' })))
  .use(queue(redisQueue(), { jobs: [resizeAvatar] }))
```

`defineConfig({ ...appOptions, bricks: [...] })` is the flat surface over the same mechanism, and
returns an `App` typed as `App` rather than carrying each contribution — a runtime array cannot
express that in the type system, which is why both surfaces exist.

> **Note.** This block used to show a `defineConfig({ db: { driver }, auth: { providers } })`
> shape that was never built. D5 settled on chained bricks with `defineConfig` as sugar over
> them; the aspirational version outlived the decision that replaced it, in this file and in the
> README, until 0.2.0.

```ts
// src/routes/users/[id]/avatar.post.ts
import { z } from 'zod'

export const auth = true
export const params = z.object({ id: z.uuid() })
export const body   = z.object({ file: z.file() })

export default async ({ params, body, user, storage, db, queue }) => {
  const { key } = await storage.upload(`avatars/${params.id}`, body.file)
  await queue.dispatch(resizeAvatar, { key })
  // `ctx.db` is the Drizzle client itself — no invented query API (D16).
  return db.update(users).set({ avatar: key }).where(eq(users.id, params.id)).returning()
}
```

That route is automatically validated, authenticated, typed, and documented in OpenAPI at
`/docs`. **That is the entire product thesis.**

### Positioning

| | Express | Fastify | NestJS | FastAPI | **Oven** |
|---|---|---|---|---|---|
| Runtime | Node | Node | Node | Python | **Bun** |
| Batteries (auth/db/queue/mail/s3) | ✗ | ✗ | partial | partial | **✓** |
| Auto OpenAPI | ✗ | brick | brick | ✓ | **✓** |
| Types inferred from schemas | ✗ | partial | partial | ✓ | **✓** |
| File-based routing | ✗ | ✗ | ✗ | ✗ | **✓** |

We are not competing on being the fastest router (Bun makes everything fast). We compete on
**time-from-`bun create` to a production-shaped app**.

---

## 2. Locked decisions

These are settled. Do not re-litigate them in a new session without the user explicitly asking.

| # | Decision | Choice | Why |
|---|---|---|---|
| D1 | Brand / scope | **Oven**, npm scope `@theoven`, CLI `oven`, docs at **theoven.app** | Unscoped `oven` on npm is taken by a dead 2013 package. Scoped packages are correct for a multi-package framework anyway. `@theoven` matches the domain exactly and carries zero naming risk. |
| D2 | HTTP core | **Own radix-tree router on `Bun.serve`** | Zero runtime deps in core, maximum speed, and total control over the Context shape — which everything else (auth, DI, OpenAPI) hangs off. A third-party router's context would leak into our public API forever. |
| D3 | Route DX | **File-based routing** (`src/routes/**`), schema + handler exported from each file | Zero wiring. The filesystem is the route table. Each file still exports its schemas, so validation, typing and OpenAPI all come from the same place. |
| D4 | Validation | **Zod 4 default, any Standard Schema v1 validator accepted** | Zod 4 has native `z.toJSONSchema()` → OpenAPI generation is free. Standard Schema means users can bring Valibot/ArkType/Effect with no lock-in. |
| D5 | Module wiring | **Brick functions + `.use()` chaining**, with `defineConfig` as typed sugar on top | Chained bricks give bulletproof type inference with a straightforward implementation. `defineConfig` expands to `.use()` calls, so the "everything is config" surface exists over a sound mechanism. Two surfaces, one core. |
| D6 | v1 scope | Core + OpenAPI, Auth (better-auth), DB (drizzle), Storage (S3), Mail, Queue, Cron | All four module groups ship in 1.0. Ambitious, deliberately chosen. |
| D7 | Repo | **Bun workspaces monorepo + changesets** | Bun is fast enough that Turbo/Nx add config without buying speed. Changesets gives independent versioning + changelogs + CI publishing. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hiteshchoudhary/theoven](https://github.com/hiteshchoudhary/theoven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->

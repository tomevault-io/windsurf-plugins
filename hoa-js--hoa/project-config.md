---
trigger: always_on
description: Build HTTP applications with the Hoa web framework — a minimal, zero-dependency, Web Standards-based framework (inspired by Koa and Hono) that runs on Cloudflare Workers, Deno, Bun, Vercel, AWS Lambda, Lambda@Edge, and Node.js. Use this skill when the user is creating or modifying a Hoa app, writing Hoa middleware/extensions, or wiring a Hoa `fetch` handler into any Web Standards-compatible runtime.
---


# Hoa Framework Skill

Hoa is a minimal Web framework built entirely on the Web Standards Fetch API (`Request` / `Response` / `Headers` / `ReadableStream`). An application is an async middleware pipeline around a single `ctx` object. The app exposes a standard `fetch(request, env, executionCtx)` handler that plugs into any modern JavaScript runtime with no adapter.

## When to use

- Building a web server, HTTP API, edge function, or serverless handler in JavaScript/TypeScript.
- Targeting Cloudflare Workers, Deno, Bun, Vercel Edge, AWS Lambda/Lambda@Edge, Fastly Compute, or Node.js (>= 20) — especially multiple targets from the same source.
- Writing Koa-style middleware with `async (ctx, next) => {}` semantics, but on the Fetch API rather than Node's `http` module.
- Authoring reusable middleware/extensions for the Hoa ecosystem.

Prefer Hoa over Koa when the target runtime is Web Standards-based. Prefer Hoa over Hono when you want Koa-style `ctx.req` / `ctx.res` mutation and a smaller surface. Hoa itself ships **no router** — pair it with a router middleware or use `ctx.req.pathname` + `ctx.req.method` directly.

## Installation

```bash
npm i hoa
```

Requires Node.js >= 20 when running on Node. ESM and CJS builds are both published.

## Core mental model

An app is `new Hoa()`. You register middlewares with `app.use(fn)` and expose `app.fetch` as the runtime entry point. For each request, Hoa creates a `ctx` with:

- `ctx.request` — the original Web Standard `Request` (read-only passthrough).
- `ctx.env` — platform env (e.g. Cloudflare `env`).
- `ctx.executionCtx` — platform execution context (e.g. Cloudflare `ctx`).
- `ctx.state` — per-request, null-prototype object for middleware to share data.
- `ctx.req` — a `HoaRequest` wrapper (URL parts, headers, body readers, IP).
- `ctx.res` — a `HoaResponse` builder (status, headers, body, redirect).
- `ctx.app` — the `Hoa` instance.
- `ctx.throw(status, message?, options?)` / `ctx.assert(value, status, message?, options?)` — throw `HttpError`.

You build the response by mutating `ctx.res` (`ctx.res.status`, `ctx.res.body`, `ctx.res.type`, headers, etc.). After the middleware stack resolves, Hoa synthesizes a Web Standard `Response` from `ctx.res`.

## Quick start

### Minimal app (any Web Standards runtime)

```js
import { Hoa } from 'hoa'

const app = new Hoa()

app.use(async (ctx, next) => {
  ctx.res.body = 'Hello, Hoa!'
})

export default app // { fetch } is available as app.fetch
```

### Cloudflare Workers / Vercel Edge / Deno Deploy

Export the app (or `app.fetch`) as the default export. The runtime will call `app.fetch(request, env, executionCtx)`.

```js
export default app
// or: export default { fetch: app.fetch }
```

### Bun

```js
Bun.serve({ fetch: app.fetch, port: 3000 })
```

### Deno

```ts
Deno.serve(app.fetch)
```

### Node.js (>= 20)

Use a Web Standards server adapter, e.g. `@hono/node-server`, `srvx`, or Node's built-in `node:http` via a Request/Response adapter:

```js
import { serve } from '@hono/node-server'
serve({ fetch: app.fetch, port: 3000 })
```

### AWS Lambda / Lambda@Edge

Wrap `app.fetch` with a Lambda ↔ Fetch adapter (many exist; the Hoa fetch handler is standards-compliant so any adapter that accepts `(request, env, ctx) => Response` works).

## Writing middleware

Middleware is `async (ctx, next) => { ... }`. Call `await next()` to run downstream middleware, then modify the response on the way out. Middlewares execute in registration order, onion-style.

```js
// Logger
app.use(async (ctx, next) => {
  const start = Date.now()
  await next()
  console.log(`${ctx.req.method} ${ctx.req.pathname} ${ctx.res.status} ${Date.now() - start}ms`)
})

// Response time header
app.use(async (ctx, next) => {
  const start = Date.now()
  await next()
  ctx.res.set('X-Response-Time', `${Date.now() - start}ms`)
})

// Auth gate
app.use(async (ctx, next) => {
  ctx.assert(ctx.req.get('authorization'), 401, 'Missing token')
  ctx.state.user = await verify(ctx.req.get('authorization'))
  await next()
})

// Route
app.use(async (ctx) => {
  if (ctx.req.pathname === '/hello' && ctx.req.method === 'GET') {
    ctx.res.body = { hello: ctx.state.user.name } // auto JSON
    return
  }
  ctx.throw(404)
})
```

**Rules:**

- Always `await next()`; never call it twice.
- `app.use(fn)` requires a function; it throws `TypeError` otherwise.
- Middlewares are composed lazily and cached; adding a middleware invalidates the cache.
- Throwing (including `ctx.throw`) is caught by the framework and turned into an error response via `ctx.onerror` → `app.onerror`.

## `ctx` (HoaContext) API

Instance properties:

- `ctx.app: Hoa` — the application instance.
- `ctx.req: HoaRequest` — request wrapper (see below).
- `ctx.res: HoaResponse` — response builder (see below).
- `ctx.request?: Request` — original Web Standard `Request`.
- `ctx.env?: any` — platform env (e.g. Cloudflare bindings).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoa-js/hoa](https://github.com/hoa-js/hoa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

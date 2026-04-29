---
trigger: always_on
description: Generates keys for every request in the `cacheName` store. This can be used to cache data based on request parameters or context parameters. The default is `c.req.url`.
---

<SYSTEM>This is the full developer documentation for Hono.</SYSTEM>

# Start of Hono documentation

# Hono

Hono - _**means flame🔥 in Japanese**_ - is a small, simple, and ultrafast web framework built on Web Standards.
It works on any JavaScript runtime: Cloudflare Workers, Fastly Compute, Deno, Bun, Vercel, Netlify, AWS Lambda, Lambda@Edge, and Node.js.

Fast, but not only fast.

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()

app.get('/', (c) => c.text('Hono!'))

export default app
```

## Quick Start

Just run this:

::: code-group

```sh [npm]
npm create hono@latest
```

```sh [yarn]
yarn create hono
```

```sh [pnpm]
pnpm create hono@latest
```

```sh [bun]
bun create hono@latest
```

```sh [deno]
deno init --npm hono@latest
```

:::

## Features

- **Ultrafast** 🚀 - The router `RegExpRouter` is really fast. Not using linear loops. Fast.
- **Lightweight** 🪶 - The `hono/tiny` preset is under 14kB. Hono has zero dependencies and uses only the Web Standards.
- **Multi-runtime** 🌍 - Works on Cloudflare Workers, Fastly Compute, Deno, Bun, AWS Lambda, or Node.js. The same code runs on all platforms.
- **Batteries Included** 🔋 - Hono has built-in middleware, custom middleware, third-party middleware, and helpers. Batteries included.
- **Delightful DX** 😃 - Super clean APIs. First-class TypeScript support. Now, we've got "Types".

## Use-cases

Hono is a simple web application framework similar to Express, without a frontend.
But it runs on CDN Edges and allows you to construct larger applications when combined with middleware.
Here are some examples of use-cases.

- Building Web APIs
- Proxy of backend servers
- Front of CDN
- Edge application
- Base server for a library
- Full-stack application

## Who is using Hono?

| Project                                                                            | Platform           | What for?                                                                                                   |
| ---------------------------------------------------------------------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------- |
| [cdnjs](https://cdnjs.com)                                                         | Cloudflare Workers | A free and open-source CDN service. _Hono is used for the API server_.                                      |
| [Cloudflare D1](https://www.cloudflare.com/developer-platform/d1/)                 | Cloudflare Workers | Serverless SQL databases. _Hono is used for the internal API server_.                                       |
| [Cloudflare Workers KV](https://www.cloudflare.com/developer-platform/workers-kv/) | Cloudflare Workers | Serverless key-value database. _Hono is used for the internal API server_.                                  |
| [BaseAI](https://baseai.dev)                                                       | Local AI Server    | Serverless AI agent pipes with memory. An open-source agentic AI framework for web. _API server with Hono_. |
| [Unkey](https://unkey.dev)                                                         | Cloudflare Workers | An open-source API authentication and authorization. _Hono is used for the API server_.                     |
| [OpenStatus](https://openstatus.dev)                                               | Bun                | An open-source website & API monitoring platform. _Hono is used for the API server_.                        |
| [Deno Benchmarks](https://deno.com/benchmarks)                                     | Deno               | A secure TypeScript runtime built on V8. _Hono is used for benchmarking_.                                   |

And the following.

- [Drivly](https://driv.ly/) - Cloudflare Workers
- [repeat.dev](https://repeat.dev/) - Cloudflare Workers

Do you want to see more? See [Who is using Hono in production?](https://github.com/orgs/honojs/discussions/1510).

## Hono in 1 minute

A demonstration to create an application for Cloudflare Workers with Hono.

![Demo](/images/sc.gif)

## Ultrafast

**Hono is the fastest**, compared to other routers for Cloudflare Workers.

```
Hono x 402,820 ops/sec ±4.78% (80 runs sampled)
itty-router x 212,598 ops/sec ±3.11% (87 runs sampled)
sunder x 297,036 ops/sec ±4.76% (77 runs sampled)
worktop x 197,345 ops/sec ±2.40% (88 runs sampled)
Fastest is Hono
✨  Done in 28.06s.
```

See [more benchmarks](/docs/concepts/benchmarks).

## Lightweight

**Hono is so small**. With the `hono/tiny` preset, its size is **under 14KB** when minified. There are many middleware and adapters, but they are bundled only when used. For context, the size of Express is 572KB.

```
$ npx wrangler dev --minify ./src/index.ts
 ⛅️ wrangler 2.20.0
--------------------
⬣ Listening at http://0.0.0.0:8787
- http://127.0.0.1:8787
- http://192.168.128.165:8787
Total Upload: 11.47 KiB / gzip: 4.34 KiB
```

## Multiple routers

**Hono has multiple routers**.

**RegExpRouter** is the fastest router in the JavaScript world. It matches the route using a single large Regex created before dispatch. With **SmartRouter**, it supports all route patterns.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kazuph) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

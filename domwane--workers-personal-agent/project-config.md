---
trigger: always_on
description: A personal agent on Cloudflare Workers, reachable from a web chat. A Durable Object holds one
---

# Personal Agent

A personal agent on Cloudflare Workers, reachable from a web chat. A Durable Object holds one
conversation and runs a tool loop against an OpenAI-compatible API; long-term memory is markdown in
R2; `evals/` is a separate Node-side harness for measuring it.

**[ARCHITECTURE.md](ARCHITECTURE.md) is the map**: the platform limits and which of them were
measured, every ceiling this agent runs into and whether it fails loudly, the invariants that caused
outages when broken, the vault layout, and what a deploy cannot provision. Read it before changing
anything under `src/`. The reasoning per subsystem is in `docs/decisions/`, indexed there.

This file is the working part: commands, how the tests are shaped, what must stay out of the
repository, and how to work here.

## Commands

```bash
pnpm test          # agent tests — Workers pool, excludes evals/
pnpm test:evals    # eval tests — Node pool
pnpm check         # tsc --noEmit for src/ and test/
pnpm check:web     # vue-tsc for web/ — a separate tsconfig, not covered by pnpm check
pnpm check:evals   # tsc for evals/ — its own tsconfig too, Node types rather than Workers
pnpm lint          # oxlint, --deny-warnings — reads .vue script blocks too
pnpm format        # oxfmt in place; format:check is the read-only form
pnpm build:web     # vite build web → ./public (gitignored; wrangler serves it)
pnpm dev           # wrangler dev — needs `cloudflared` once Access is on the account, see below
pnpm dev:web       # vite with hot reload, proxying /agents and /api to :8787
pnpm run deploy    # wrangler deploy — User runs this, not the agent; bare `pnpm deploy` is pnpm's builtin
```

`./public` is build output and gitignored. `wrangler.jsonc` names `pnpm build:web` as the custom
build, so `pnpm dev` and `pnpm run deploy` both produce it; run it by hand only outside wrangler.

**`pnpm dev` needs `cloudflared` and a terminal once Cloudflare Access is on the account**
(2026-08-23). The `ai` binding has no local simulator, so dev opens a remote proxy session against
the deployed Worker, which Access gates. `brew install cloudflared` once, and then wrangler sends
you to the Access login.

The second half is the one that costs an hour: **it must be interactive.** Piping dev through `tee`
is enough to make it non-interactive, and it then fails with *"no Access Service Token credentials
were found and the current environment is non-interactive"*, which reads as a credentials problem
and is a TTY problem. `script -q /tmp/dev.log pnpm dev` keeps both. For a genuinely non-interactive
run, an Access **service token** in `CLOUDFLARE_ACCESS_CLIENT_ID` / `CLOUDFLARE_ACCESS_CLIENT_SECRET`
is the supported path.

`pnpm test` and `pnpm test:evals` are **two different pools and neither runs the other's tests**.
Running only one and declaring the suite green is a mistake that has happened.

**Lint and format decisions live in [docs/decisions/linting-and-formatting.md](docs/decisions/linting-and-formatting.md)**:
which rules are on, which were counted and refused, and why type-aware linting does *not* need the
TypeScript 7 upgrade that would break `vue-tsc`. One thing to know without reading it: `pnpm lint`
runs type-aware rules with no extra flag.

Eval scripts are `eval:*` in `package.json` and run raw TypeScript through Node with no build
step (`node evals/run-retrieval.ts`). Anything under `evals/` may use `fs`, `process`, and the
network; anything under `src/` may not.

## Tests

The agent suite runs in `@cloudflare/vitest-pool-workers` with isolated storage. Two traps:

- **`fs` and `process` do not exist in the Workers pool.** That is why `evals/` has its own config.
- **An alarm scheduled by a test can fire after teardown**, producing
  `Isolated storage failed ... Application called abortAllDurableObjects()`. If a test causes code
  to `schedule()`, assert the schedule exists, cancel it, and invoke the callback directly rather
  than waiting for it.

Outbound HTTP is intercepted with `fetchMock`; `assertNoPendingInterceptors()` catches mocks that
stopped matching after a change. When a change alters call *counts*, for example an index
reconcile that now reads fewer files, the interceptor counts must be updated, not the assertion
removed.

The vault is **not** mocked over HTTP. Miniflare simulates R2 locally, so tests seed the real
binding through [test/helpers/vault.ts](test/helpers/vault.ts) and assert on what the bucket
holds. A file that is simply absent is the not-found case and needs no setup at all.

Bindings are pinned in `vitest.config.ts` (`LLM_BASE_URL`, `LOG_CONTENT: ''`) so the suite never
depends on the real values in `wrangler.jsonc`. `LOG_CONTENT` is the one this caught: the
deployment opts in, and an inherited binding made a test assert content logging was off while it
was on.

Two more, both found the hard way:

- **`SELF.fetch` cannot see a mutated `env`.** The pool hands the test a copy, so a test that needs
  a different binding (the Cloudflare provider branch, where `LLM_BASE_URL` is *absent* while the
  suite pins it) must call `worker.fetch(request, {...env, LLM_BASE_URL: undefined}, ctx)` directly.
- **The Cache API outlives a test's mocks.** `/api/models` cached in it once, and one test's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DomWane/workers-personal-agent](https://github.com/DomWane/workers-personal-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->

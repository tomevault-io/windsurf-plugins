---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

`jean-claude` is a CLI MITM HTTPS proxy: it intercepts another tool's API traffic
and rewrites it from a YAML file. See `README.md` for user-facing behaviour and
the config format — this file covers how to work on the code.

**Everything in this project is written in English**: code, comments, error
messages, docs, commit messages, test names.

## Commands

```bash
npm test                 # vitest run - 155 tests, ~1.6s
npm run test:watch
npm run typecheck        # tsc --noEmit
npm run lint             # eslint
npm run format           # prettier --write
npm run build            # tsdown -> dist/cli.mjs
npm run jc -- --help     # run the CLI from source (tsx)
node src/cli.ts --help   # also works: Node 24 strips types natively
```

**Use `npx vitest run` here, not `npx ng test`.** The sibling projects in
`~/RelyensProjects` are Angular apps where `ng test` is mandatory; this one is a
plain Node CLI and has no Angular builder.

## Architecture

```
src/
  cli.ts              commander wiring: run | start | env | ca | check | init
  commands/
    shared.ts         openSession(): config + CA + upstream + proxy + hot reload
    run.ts            execa spawn with the injected env; exits with the child's code
    start.ts          foreground daemon; writes the session file
    env.ts            prints the env for an already running start
    ca.ts check.ts init.ts
  config/
    schema.ts         zod schema - THE source of truth for the config format
    paths.ts          the jean-claude home and everything under it
    load.ts           YAML read, validation, rule compilation
    watch.ts          chokidar -> reload
    template.ts       `init` scaffolding (generic + --claude-code)
  proxy/
    server.ts         getLocal(), lifecycle, listeners, rule registration
    actions.ts        respond / patch / request -> mockttp callbacks
    match.ts          host/method/path/query -> predicate (mockttp-free, unit tested)
  ca/store.ts         CA generation + bundle.pem assembly
  env/
    upstream.ts       reads the inherited HTTPS_PROXY / NODE_EXTRA_CA_CERTS
    child.ts          builds the env injected into the target tool
    session.ts        session.json handoff between `start` and `env`
  record/writer.ts    --record: writes real responses as reusable stubs
  log/
    reporter.ts       one console line per request, plus the exit summary
    sink.ts           --log: sends our own output to a file while a child runs
  util/json.ts        deep merge + RFC 6902 JSON Patch
```

### How rules become mockttp rules

`proxy/server.ts` registers **one mockttp rule per config rule**, in file order,
then a `forUnmatchedRequest()` catch-all. Matching goes through
`.matching(compileMatcher(...))` rather than mockttp's own URL matchers, so all
matching logic lives in `proxy/match.ts` and stays testable without a server.

Actions use mockttp's **imperative callbacks** (`thenCallback`, `beforeRequest`,
`beforeResponse`), not its declarative `transformRequest`/`transformResponse`.
That was a deliberate trade: the declarative path is more idiomatic but is a
static per-rule option, which rules out `delay` on a patched response and forces
two code paths. One imperative path handles every mode uniformly.

`beforeRequest` is installed even on rules that do not rewrite anything, because
that is where a request gets tagged for the log.

## mockttp API notes

Pinned to `mockttp@^4.6.1`. Things that cost time to discover:

- **Callback types are not re-exported from the package root.** `actions.ts`
  derives them from the public `RequestRuleBuilder` type
  (`Parameters<RequestRuleBuilder['thenPassThrough']>[0]`) rather than deep
  importing. Keep it that way — it survives version bumps.
- **`reset()` clears rules _and_ subscriptions.** Hot reload must re-attach the
  `on('request'|'response'|'abort'|'tls-client-error')` listeners after every
  reset. `test/integration.spec.ts` has a test that fails if this regresses.
- **`getCA` is not on the root export** but mockttp's `exports` map publishes
  `./dist/*`, so `import { getCA } from 'mockttp/dist/util/certificates'` works
  (no `.js` suffix). Used in tests only, to mint the fake upstream's leaf.
- `additionalTrustedCAs`, not `trustAdditionalCAs`. `proxyConfig` takes
  `{ proxyUrl, noProxy }`.
- `tlsPassthrough` is a **server construction** option, so changing it needs a
  restart. `reload()` warns when it changes.

## Testing

`test/integration.spec.ts` is the one that matters. It drives the real
`startProxy()` against a fake upstream and asserts the properties that define
correctness:

- a `respond` rule **never reaches the server** (`upstreamHits` stays empty)
- a `patch` rule **does** reach it
- a status-only patch **leaves the body intact** (regression guard: a partial
  callback result used to risk blanking it)

The fake upstream is a plain `node:https` server holding a leaf minted by
jean-claude's own CA. **Do not replace it with a second mockttp instance**: a
mockttp proxy relaying to another mockttp instance stalls ~15.1 s on the first
upstream TLS connection. It is not `http2`, not certificate validation, not DNS,
and not jean-claude — against a real server the relay costs ~25 ms. Using

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EtiennePasteur/jean-claude](https://github.com/EtiennePasteur/jean-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

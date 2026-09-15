---
trigger: always_on
description: You must understand and be able to explain any code you write. If you cannot explain
---

# AGENTS.md — rules for any agent working on this repo

## The One Rule

You must understand and be able to explain any code you write. If you cannot explain
why a line exists, delete it or learn why before shipping. (Mirrors pi's own
CONTRIBUTING "One Rule" — we apply it to ourselves too.)

## No fabricated model metadata

Every context-window, max-token, modality, cost, and compat value must trace to a
source:

- **models.dev** (provider `nan` in `https://models.dev/api.json`) — the default
  source, pulled by `scripts/generate-models.ts`; or
- **an explicit manual note** recorded on the generated entry
  (`notes` in `scripts/models.generated.ts`) stating where the value was confirmed
  (URL + date).

Never guess limits. If a model is missing from models.dev or has incomplete limits,
the generator omits it and flags it (`needs manual verification`); do not invent
numbers to fill the gap. The same applies to auth mechanics: only documented pi
behavior (`docs/custom-provider.md` shipped with pi) — no invented flows.

## Verify before done

Run both before considering any task done:

```bash
bun test        # all tests must pass
bun run typecheck  # typecheck must be clean (bunx resolves tsc; bun publish lifecycle lacks node_modules/.bin on PATH)
```

Regenerate the catalog after touching `scripts/generate-models.ts`:

```bash
bun run generate-models
```

## Live NaN API during diagnosis

Diagnostic calls against the real gateway are allowed — they spend the
maintainer's quota, so they are **permission-gated**:

- **Ask the maintainer before running any live probe, with an approximate token
  cost (input + output).** No silent probing. If the cost is not worth it, report
  the behavior to NaN and let them reproduce it instead of debugging it here.
- **Tests must never hit the network.** `bunfig.toml` preloads
  `test/network-guard.ts`, which makes any un-injected `fetch` throw. Keep it:
  inject `fetchImpl` / `options.fetch`, or use the local fixture. The permission
  gate covers ad-hoc diagnosis only, never `bun test`.
- **Default to `qwen3.6` — it is unlimited.**
- **For massive/bulk probes prefer a model the maintainer uses less with a large
  token budget, e.g. `mimo-v2.5`** (1M context).
- **When the model under investigation is the point** (e.g. reproducing a
  model-specific 400), use it, but minimize tokens: smallest viable prompt,
  lowest `max_tokens`, stop at the first decisive response.
- Repro commands that run a real `pi` session (`pi --fork ... -p ...`) use the
  same key; keep them minimal and delete the forked session files afterwards.

## One shared implementation for all providers

`nan` (and any future provider, e.g. `helmcode`) must stay behind the single shared
factory in `src/provider-factory.ts`. A second provider-specific file is a smell:
refactor back to the factory and add a config entry in `src/providers.ts` instead.
The `factory is shared` test in `test/provider-factory.test.ts` guards this contract.

## Version policy (strict semver)

Every PR that changes code MUST bump `package.json` version in the same PR; CI publishes only when the version differs from npm.

- **PATCH** (`0.1.z`): bug fixes, docs, comment-only changes, catalog regeneration with identical values.
- **MINOR** (`0.x.0`): new features — new provider entries, new MCP tools, new env vars/config options, and (while `0.x`) breaking changes, each breaking change called out explicitly in the PR/changelog.
- **MAJOR** (`x.0.0`): breaking changes once `1.0.0` is reached.
- Never reuse a published version; never publish with failing tests (CI gates publish on tests + typecheck).
- The npm registry is the source of truth for "published"; `.github/workflows/publish.yml` compares `package.json` against `npm view` and publishes only on difference.

## Verified API facts (do not re-derive from stale docs)

- **Extension-side pi-ai imports (v0.5.0, verified on pi-ai 0.83.0 AND 0.84.4):**
  statically import ONLY the bare `@earendil-works/pi-ai` root from `src/`. pi's
  extension loader maps that specifier to the compat entrypoint in every loading
  mode (bundled CLI interception, Node-mode jiti aliases, compiled-binary
  virtualModules), and the compat entrypoint re-exports every lazy API factory —
  including `openAICompletionsApi`. A static SUBPATH import
  (`@earendil-works/pi-ai/api/...`) gets the alias applied as a prefix and
  resolves to `<compat.js>/api/...`, which does not exist: the whole extension
  fails to load (the v0.4.x load failure). Type-only subpath imports are erased
  before resolution and are safe; a DYNAMIC subpath `import()` is the sanctioned
  plain-node fallback and never runs under pi because the root (compat) exports
  the factory. Guarded by `test/extension-load.test.ts`.
- The REAL pi-ai root (plain node/bun, outside pi) does not export
  `openAICompletionsApi`; `createProvider` and `envApiKeyAuth(name, envVars)` are
  on the root. `envApiKeyAuth` implements exactly: stored credential key wins →
  first set env var → unconfigured; `login()` prompts with `{ type: "secret" }`.
- pi awaits extension factories (`await factory(api)`) on 0.83.0 and 0.84.4
  alike, so the extension entrypoint may be async (v0.5.0: streaming-API
  resolution needs it).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gtrabanco/pi-nan-provider](https://github.com/gtrabanco/pi-nan-provider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->

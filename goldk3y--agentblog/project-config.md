---
trigger: always_on
description: A shadcn registry, a CLI, and a Claude Code plugin that install a production
---

# AgentBlog

A shadcn registry, a CLI, and a Claude Code plugin that install a production
Next.js blog which AI answer engines can actually read.

The characteristic failure mode of everything in this repository is **silence**.
A broken install compiles, lints, renders perfectly in a browser, and is invisible
to the crawlers the whole product exists for. That is why `scripts/` holds
eighteen assertion gates and why most rules below are absolute rather than
stylistic. When you are unsure whether something works, reach for `curl` and a
script, not a screenshot.

`CONTRIBUTING.md` has the long form of every rule here. `DEPLOYMENT.md` covers
publishing and the Vercel deploy.

## Environment

- pnpm 11, Node 20.9+ (CI runs 22). Never `npm` or `yarn`.
- Install with `pnpm install --frozen-lockfile`. A plain `pnpm install` rewrites
  the lockfile silently, which looks fine locally and stops every CI job at the
  install step.

## Commands

```bash
pnpm codegen                              # after touching any source of truth (rule 2)
pnpm --filter @agentblog/web dev          # the dogfood site; bare `pnpm dev` runs every app
pnpm --filter @agentblog/docs dev         # docs.agentblog.dev, on port 3001
pnpm typecheck && pnpm lint && pnpm test
pnpm format                               # CI runs format:check

pnpm check:static                         # every check that needs no build. Run before pushing.
pnpm --filter @agentblog/web registry:build && pnpm check:registry

pnpm turbo run build --filter=agentblog   # build the CLI
```

Build the CLI through turbo, not `pnpm --filter agentblog build`. tsup bundles
`@agentblog/checks` and `@agentblog/schema` and resolves both through their
`dist`, so a direct filter runs tsup with nothing built and esbuild fails to
resolve them.

Single test file: `node --test packages/cli/src/patchers/patch-set.test.ts`.
`@agentblog/schema` additionally needs `--experimental-strip-types`, and
`@agentblog/web` needs `--import ./tests/alias-hook.mjs`.

## Layout, only the parts you cannot infer

- **`apps/web/registry/**` is the source of truth for every file AgentBlog ships
  into a user's project.** It is authored as if it were already there: same
  `@/components/...` and `@/lib/...` aliases, same directory shape. That is why
  shipped code lives inside the web app instead of in `packages/`.
- **The routes under `apps/web/app/blog/**`, `app/authors/**`, `app/sitemap.ts`
  and friends are thin re-export shims** over that registry source, so the demo
  site is the same modules the registry ships. Edit the registry file, not the
  shim. Route segment config (`export const revalidate`) is the one thing Next.js
  refuses to let you re-export, so it is duplicated in both and has to be kept in
  sync by hand.
- **`apps/docs` is docs.agentblog.dev**, a Fumadocs site whose pages are MDX
  files under `apps/docs/content/docs`. A page's URL is its path, sidebar order
  comes from the `meta.json` beside it, and `title` and `description` are both
  required. `agentblog.dev/docs/*` 301s here, page by page, from
  `apps/web/next.config.ts`.
- `packages/` holds what does not ship as source: `schema` (Zod schemas, inferred
  types, the `ContentSource` contract suite), `checks` (config predicates),
  `cli` (the `agentblog` npm package), plus the shared eslint and tsconfig bases.
- `IMPLEMENTATION.md` and `blog-architecture-guide.md` are gitignored local
  planning documents. Do not commit them, and do not quote them in shipped prose.

## Rules

### 1. Never use an em dash

Not in code comments, docs, registry `docs` strings, seed MDX, CLI output, or
commit messages. Use a comma, a colon, parentheses, or a full stop and a new
sentence. The same pass bans "delve", "leverage", "robust", "seamless",
"landscape", "tapestry", "in today's fast-paced world", the "it's not just X,
it's Y" construction, rhetorical-question-then-answer openers, and three-item
lists where two would do.

The reason is commercial, not aesthetic. The seed posts are the format
specification, so one tell in a seed post is one tell in every post that install
ever produces. `node scripts/assert-copy-style.mjs` enforces it in CI.

### 2. Generated files are generated. YOU MUST NOT edit them in place.

| Generated                                        | Source of truth                        |
| ------------------------------------------------ | -------------------------------------- |
| `apps/web/registry/blog/lib/schemas.ts`          | `packages/schema/src/schemas.ts`       |
| `apps/web/registry/blog/lib/types.ts`            | `packages/schema/src/types.ts`         |
| `apps/web/registry/blog/lib/define-config.ts`    | `packages/schema/src/define-config.ts` |
| `apps/web/registry/blog/lib/preflight-checks.ts` | `packages/checks/src/core.ts`          |
| `plugins/agentblog/skills/**`                    | `apps/web/registry/agent/skills/**`    |

Edit the source, run `pnpm codegen`. CI runs `pnpm codegen:check` and fails on
drift. The skills are a copy rather than a symlink on purpose: Claude Code copies
a plugin directory to a cache location on install, so a symlink pointing outside
that directory ships empty skills, which looks fine locally and breaks only for
users.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goldk3y/agentblog](https://github.com/goldk3y/agentblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

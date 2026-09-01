---
trigger: always_on
description: File-hosting backend for **uploads.sh**. Provider-agnostic storage via
---

# uploads

File-hosting backend for **uploads.sh**. Provider-agnostic storage via
[files-sdk](https://files-sdk.dev), deployed to Cloudflare Workers with
Wrangler. The service is public: anyone can sign up, create a workspace, and
use the CLI, and paid plans are live.

## Layout

The path-by-path inventory lives in the README's
[What's in this repo](README.md#whats-in-this-repo) table — one copy, so it
can't drift. `packages/uploads` also ships `uploads mcp`, a stdio MCP server
mirroring the CLI commands.

Three agent skills are checked in at the repo root so they're installable via
the `npx skills add` convention (and by `uploads install`):
`skills/github-screenshots` is the thin workflow skill (when a screenshot or
recording should go into a PR/issue or be shared as a link — the in-repo
successor to the external `github-screenshots` skill's bundled R2 scripts),
`skills/annotate-screenshots` covers callouts and redaction
(`uploads annotate` / `screenshot --annotate`), and `skills/uploads-cli` is
the full CLI reference the others defer to. Keep all three in sync when the
CLI's commands or flags change.

**Screenshots: stage as you go.** If your change is visually observable (web
UI, email templates, rendered output), capture and stage screenshots at each
milestone while you work — don't wait for a PR to exist. A bare `uploads put`
already stages by default on a non-default git branch (issue #403), and a
bare `uploads screenshot` (no `--pr`/`--issue`/`--branch`) does the same
(issue #469) — carrying its derived metadata (`path`/`url`/`env`/`viewport`,
plus `--state`) through to the PR once it opens; reach for `attach --branch`
when you want its extras (multiple files in one call, or triggering
promotion/comment sync as a side effect):

```bash
uploads put ./after.png --meta path=/settings --state after   # before|after|empty|error|loading
uploads screenshot http://localhost:4321/settings --out after.png --state after
uploads attach ./after.png --branch --state after              # explicit form, either way
```

Opening a PR automatically promotes everything staged for the branch into one
managed comment (via the GitHub App webhook, or the next `uploads attach` /
`uploads attach --promote` without it). See `skills/github-screenshots` for
the full workflow.

Keep API and web separate deployables. All storage access goes through
`createStorage()` in `packages/storage` — never import files-sdk adapters or
touch the R2 binding directly from route code. Adding a provider = a new case
in `createStorage` plus its files-sdk peer deps.

## Commands

```bash
pnpm bootstrap           # one-command local setup (tooling, deps, env, types, D1, default workspace)
pnpm doctor              # read-only diagnose of the local setup
pnpm install
pnpm dev                 # API on :8787 (local R2 + KV + D1 simulation)
pnpm dev:web             # Astro site
pnpm typecheck           # wrangler types + tsc across workspaces
pnpm test                # whole suite in one vitest process (all packages); CI's Test job runs this
pnpm test:api            # single package (also test:mcp / test:auth / test:web / test:cli;
                         # uses vitest defaults, not the root config)
pnpm build:cli           # build @buildinternet/uploads without the --filter incantation
pnpm run deploy          # all workers; or deploy:api / deploy:web / deploy:mcp
pnpm workspace:add <name> [--bucket <bucket>] [--binding X] [--local] \
  [--no-default-limits] [--max-storage …]   # shared/agent limit template by default
pnpm workspace:limits <name> [--max-storage …] [--max-video-bytes …] \
  [--allowed-prefixes default|f,screenshots,gh] [--max-key-depth 8] \
  [--clear-max-storage] [--clear-allowed-prefixes] […]
pnpm migrate:d1:local    # apply apps/api/migrations to local D1 (migrate:d1 = remote)
pnpm uploads put <file> --env-file .env   # monorepo only: builds package first
pnpm uploads put <file> --pr <num>
```

**CLI examples — installed binary vs monorepo:** product-facing examples
(PR “how to try it”, skill docs, issue comments, user-facing README snippets)
use the global binary as someone who already installed the CLI would:

```bash
uploads put ./shot.png
uploads put ./after.png --pr 123
```

Reserve `pnpm uploads …` for **in-repo** development (build-from-source via the
root script). Do not assume readers have the monorepo checked out.

Use `pnpm run deploy` (not bare `pnpm deploy` — that's pnpm's built-in).
`deploy:api` applies pending D1 migrations (`migrate:d1`) before
`wrangler deploy`. On merge to main, `.github/workflows/d1-migrations.yml`
also applies remote migrations when `apps/api/migrations/**` changes
(secrets: `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`). Production
worker deploys normally happen via Workers Builds on push to main. For an
isolated branch environment, `npx wrangler preview` from an app directory —
playbook in [docs/previews.md](docs/previews.md); the Workers Builds bot's
"Branch Preview URL" on PRs is a different, legacy mechanism that runs
against production bindings.

Operator runbook: [docs/ops.md](docs/ops.md). Daily retention cron on the API
worker; BYO secrets use `WORKSPACE_SECRETS_KEY`, set on both `uploads-api` and
`uploads-mcp`; bare upload keys get `f/<id>/…`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buildinternet/uploads](https://github.com/buildinternet/uploads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->

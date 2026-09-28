---
trigger: always_on
description: Canonical instructions for coding agents and humans: the durable invariants
---

# Agent Guidelines — growth.engineer

Canonical instructions for coding agents and humans: the durable invariants
and the routing table into the deep-dive docs. CI caps it at 200 lines
(`pnpm docs:check`) — one canonical statement per policy, no changelog.

## What this is

An open-source catalog of **companies**, the **tools** they make, and
**workflows** that put tools to work. A COMPANY makes many TOOLS; a tool is
ONE function an agent can call (`apollo/enrich-person`), tied to a specific
MCP tool, CLI subcommand or API endpoint — not the product. A WORKFLOW is
several tools in order with the instructions that reach a result, and a growth
hack IS a workflow, not a second kind. **Every tool and workflow is ONE
generated markdown file any agent can run; copying it is the product action.**
Reads are public; agents fetch files with no sign-in.

**THE CATALOG IS THE REPOSITORY.** Every entry is a markdown file under
`companies/` and `workflows/`, plus the vocabulary in `tags.yml`; the site is
built from them, and the community contributes by pull request.

## Changing the catalog

Adding or fixing a company, tool, workflow or tag touches only `companies/`,
`workflows/` and `tags.yml`, then `pnpm content:check`. Start at
[`CONTRIBUTING.md`](CONTRIBUTING.md); every field is in the folder READMEs
([`companies/`](companies/README.md), [`workflows/`](workflows/README.md)),
and two skills do it end to end:
[`add-workflow`](.agents/skills/add-workflow/SKILL.md) and
[`research-company`](.agents/skills/research-company/SKILL.md). Never edit a
rendered file or the app to change a fact. The rest of this file is for
changing the site itself.

## Stack

Next.js 16 (App Router, Cache Components, Turbopack) · a build-time content
compiler (`lib/content/`) · Tailwind v4 · shadcn on Base UI · Biome · Vitest
· pnpm. **The catalog has no backend, no database and no auth provider**;
the one runtime store is an optional Upstash Redis counting workflow copies
(`lib/usage/copies.ts`: Uses, Hot, Popular). Every route is public; every
env var is optional (`.env.example`, read only through `lib/env.ts`).

## Validation — proportional, not ceremonial

- **While editing**: `pnpm exec biome check --write <all touched files>` once
  per unit of work, in ONE call (every invocation loads the whole project).
  Plus `pnpm test:run tests/<exact file>` for the behavior you touched.
- **Touched catalog data** (`companies/`, `workflows/`, `tags.yml`):
  `pnpm content:check` — every problem, with its file path.
- **Once per unit of work**: `pnpm check` (Biome + `tsgo`). Not per patch.
- **Final handoff**: `pnpm tsc` then `pnpm lint`. Touched the renderer: the
  goldens in `tests/render-markdown.test.ts` must still pass byte for byte.
- **Docs only**: `pnpm docs:check`.

Heavy commands (`check`, `tsc`, `build`, `test:run`, `knip`) queue through
`scripts/heavy-lock.mjs`, one at a time per repository; a lock timeout is a
queue timeout, not a failure. `content:check` is light and runs at once. Never call `vitest`, `tsc`,
`next build` or `knip` directly; dev servers are `pnpm dev`. What CI blocks
on: [`docs/maintainers/ci.md`](docs/maintainers/ci.md).

## Critical invariants

### The markdown file

- ONE render path: [`lib/catalog/render-markdown.ts`](lib/catalog/render-markdown.ts)
  (and `render-tag.ts` for tags; pure), called only by
  [`lib/content/build-documents.ts`](lib/content/build-documents.ts) at build time. Nothing renders on the request path; a rendered file is
  never hand-edited. A SOURCE file is a YAML header of facts; a company or
  workflow adds a markdown body (a tool file is its header alone): a
  workflow's inputs, steps and checks are body sections ([`lib/content/workflow-body.ts`](lib/content/workflow-body.ts));
  the build adds setup and rules.
- The format is the contract in [`docs/markdown-files.md`](docs/markdown-files.md):
  flat YAML header, setup picks the best way in (official MCP → CLI → API →
  community; tool files list every option, workflow files ≤ 2 per tool, each
  company's ways once), inputs in backticks, ≤ 10 steps, Rules last and
  immutable, tool ≈ 80 lines, workflow ≈ 200. Change the format and the
  golden fixtures in `tests/fixtures/markdown/` in the same commit.
- A rendered file's `updated` is the newest `updated` among the source files
  that fed it (tool ← company, workflows; workflow ← tools, their companies).

### Keys and refs

- Public identity is the `key` (`apollo`, `apollo/enrich-person`,
  `funding-signal-outbound`), and the key IS the path:
  `companies/apollo/`, `companies/apollo/tools/enrich-person.md`,
  `workflows/funding-signal-outbound.md` (FLAT — no folders; the workflow's
  `author` is a GitHub login in its header, never a company). Keys are never
  authored in a header.
  Grammar and reserved handles live in [`lib/catalog/keys.ts`](lib/catalog/keys.ts);
  every top-level route must be reserved (pinned by `tests/keys.test.ts`).
- Keys never change after publishing. A rename lists the old key under
  `aliases:`; every miss asks the alias map before answering 404, and the
  `.md` handler and the pages turn a hit into a real 308.
- Deprecated stays visible with a warning; a `draft` tool or workflow has no

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GetBrew/growth-engineer](https://github.com/GetBrew/growth-engineer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->

---
trigger: always_on
description: This is the source for `seo`, a local-first TypeScript SEO CLI, library,
---

# Agent Notes

This is the source for `seo`, a local-first TypeScript SEO CLI, library,
router skill, and stdio MCP server. The public repository is
`iannuttall/seo`. The public npm package is the unscoped `seo` package.

`PRODUCT.md` is the durable product definition: users, purpose, brand
personality, vocabulary, and anti-references. Read it before writing any
user-facing copy or making product-shape decisions. This file owns the
engineering contract.

`CONTENT.md` is the durable writing guide for website copy, documentation,
report pages, command help, onboarding, metadata, and README content. Read it
before writing or editing user-facing language.

Reading `CONTENT.md` is mandatory in the same task before writing, editing, or
generating user-facing copy. Do not rely on memory or treat shared template
copy as an exception.

The product has two primary users:

- Humans start with a guided prompt flow and one main report.
- Agents use explicit flags, structured JSON, the router skill, and MCP tools.

Keep the human path calm. Keep the agent path powerful. Both paths must use the
same core report logic and return the same evidence.

`CLAUDE.md` must stay a symlink to this file. Do not maintain separate copies of
agent instructions.

## Product Direction

- The product is local first. Do not add a required hosted backend, account,
  database, job queue, or telemetry service.
- Reports, tokens, project profiles, and caches stay on the user's machine.
- A future hosted API or remote MCP may live in this monorepo and deploy to
  Cloudflare, but local CLI and library use must remain first class.
- Report accuracy, deterministic output, and simple onboarding matter more than
  adding another surface or speculative score.
- The moat is the `seo` package, product quality, verified OAuth app, test
  corpus, report depth, brand, and release velocity. Do not hide core report
  logic behind private packages.
- The license is Apache-2.0. Brand and trademark rules belong in a separate
  public policy, not in code-level restrictions.

## Public Package Contract

The repository is a monorepo internally, but users install one package:

```txt
seo package     core TypeScript API
seo/mcp         stdio MCP server API
bin: seo        executable CLI command
skills/         packaged agent skills
```

- Do not publish or teach `@seo/core`, `@seo/cli`, or `@seo/mcp`.
- The root `package.json`, `tsdown.config.ts`, and `scripts/package.test.mjs` own
  the public package contract.
- Runtime bundles must not depend on private workspace package names.
- Keep Node 22 or newer as the supported runtime unless the whole repository is
  deliberately migrated and verified.
- The product name is SEO as a wordmark, `seo` as the command and package,
  and SEO Skill when prose truly needs a name; see PRODUCT.md's Name
  section. Speak of the skill in the singular; plural "skills" survives
  only in ecosystem names like the `skills/` directory and `npx skills
  add`. Prose defaults to benefit-first copy that does not name the product
  at all. The tagline is "The SEO command for AI agents". Do not use
  "SEO Skills CLI" or "SEO CLI" in new copy; they survive only as JSON-LD
  alternate names.
- Teach `npm i -g seo`, then `seo start`, as the primary README path.
- Library and contributor setup belongs below normal CLI usage in the README.

## Repo Map

- `packages/core`: report logic, storage, providers, Search Console/Google Analytics clients,
  fetch/extract, crawling, analysis, workflows, and renderers.
- `packages/cli`: `seo` command, prompt flows, command help, selection, and
  terminal output.
- `packages/mcp`: local stdio MCP server exposing core analysis.
- `apps/web`: static Astro documentation and landing site for seoskill.dev.
- `skills/seo/SKILL.md`: the single router skill agents install. It teaches
  discovery, the jobs table, and evidence rules; per-report depth lives in the
  registry.
- `evals/`: behaviour evals keyed by report id or job, shipped in the package.
- `scripts`: package, release, quality, OAuth injection, and local utilities.
- `docs`: git-ignored local notes, audit evidence, and working material.
- Working plans live in the git-ignored `docs/plans/` directory. Never depend
  on local docs or plans for product behavior or durable context.
- `dist`: generated public package bundles. Do not hand-edit or commit them.

Changes anywhere under `apps/web` must also follow `apps/web/AGENTS.md`. Read
that file before inspecting, editing, or generating site code or copy.

Useful web areas:

- `apps/web/src/layouts/BaseLayout.astro`: canonical SEO, metadata, header, and
  footer contract.
- `apps/web/src/pages`: landing, docs, policy wrappers, and the custom 404.
- `apps/web/scripts/build-sitemap.mjs`: exact static sitemap generation.
- `apps/web/AGENTS.md`: site content, design, and deployment rules.

Useful CLI areas:

- `packages/cli/src/index.ts`: root command registration and curated help.
- `packages/cli/src/args.ts`: shared argument parsing, including `projectArg`.
- `packages/cli/src/selection.ts`: project, site, and Google Analytics selection.
- `packages/cli/src/commands/setup`: `seo start` and guided onboarding.
- `packages/cli/src/commands/mcp-clients.ts`: MCP client paths and detection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iannuttall/seo](https://github.com/iannuttall/seo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

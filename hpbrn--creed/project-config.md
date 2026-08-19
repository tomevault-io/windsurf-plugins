---
trigger: always_on
description: You're an AI coding agent picking up the Creed codebase. This file is the
---

# AGENTS.md

You're an AI coding agent picking up the Creed codebase. This file is the
short version of `README.md` + `CONTRIBUTING.md` written for you.

If a human is reading this, the document you want is [`README.md`](./README.md).

---

## What Creed is

One personal context profile every AI reads before answering the user.
10 sections (5 always-on, 5 optional). Plain Markdown content. Connected
agents read it and propose updates; users approve.

Creed is **not** a notes app, journal, chat memory store, or generic AI
wrapper. If a change would make it feel like one of those, it's the
wrong change.

---

## Stack

```
Next.js 16 (App Router, Turbopack)   React 19   TypeScript (strict)
Tailwind v4   shadcn/ui   Tiptap   Framer Motion / motion
Supabase (Postgres + RLS + auth)   OpenRouter (credits + BYOK)
```

---

## Repo layout

```
apps/
├── open/                     thin self-hosted Next.js composition
├── cloud/                    thin managed Next.js composition
├── docs/                     independent docs.creed.md Next.js app
└── status/                   independent status.creed.md Next.js app
packages/
├── creed-app/                shared product, marketing, routes, and AI
├── creed-open/               Open owner access and Open-only compositions
├── creed-cloud/              accounts, billing, Shared, and Cloud routes
├── creed-core/               domain types and pure Creed logic
├── creed-ui/                 reusable interface primitives
├── persistence/              shared Supabase clients
└── integrations/             protocol and integration helpers
.agents/
├── context/            versioned internal context pack (read this first)
└── skills/             task-triggered agent workflows
```

Unless a path explicitly starts with `apps/` or `.agents/`, shared application
paths in this file are relative to `packages/creed-app/`.

The four "god" files to be careful in:
- `packages/creed-app/components/creed/file-screen.tsx`, the editor
- `packages/creed-app/lib/creed-backend.ts`, shared Supabase glue
- `packages/creed-core/creed-data.ts`, types, agent contract, and seed
- `packages/creed-app/components/creed/settings-screen.tsx`, settings tabs

---

## Reading order before edits

1. [`lucidity.md`](./lucidity.md)
2. `.agents/context/index.md`
3. The task-relevant files in `.agents/context/` listed by `index.md`
4. The exact code path you're about to change

If `.agents/context/` is unexpectedly missing, read `lucidity.md` +
`README.md` + `CONTRIBUTING.md` + `SECURITY.md` and then this file
end-to-end.

---

## Repository skills

When a skill is delivered as `/name` or `$name`, treat it as an explicit request
to load that skill. A direct natural-language request for the same operation is
equivalent. Slash-command support varies by client, so skills must never depend
on `/name` as their only invocation path. Invocation loads the workflow but does
not bypass its permission, confirmation, or safety gates.

- Before creating any Git commit, always read and apply
  `.agents/skills/tasks/commit/SKILL.md`, even when the current agent does not
  discover repository skills automatically.
- Before opening or updating a GitHub pull request, always read and apply
  `.agents/skills/tasks/pr/SKILL.md`. The PR title is the squash commit that
  will land on the base branch. Product-release titles use `release open 1.0.0`.
  The body is plain prose, not a template. Do not use Summary or Test plan
  headings. The skill does not grant merge, tag, version, or publish authority.
- Before an intentional Open, Cloud, CLI, or Bench product release, always
  read and apply `.agents/skills/tasks/semver/SKILL.md`. A commit targeting
  `main` is not automatically a product release. The skill owns SemVer,
  release metadata, and release copy; it does not grant commit, tag, push, or
  publication authority. The status site is not a versioned product.
- Read and apply `.agents/skills/tasks/comment/SKILL.md` whenever adding,
  rewriting, or auditing source-code comments. Comments must explain durable,
  non-obvious intent rather than narrate syntax or preserve implementation
  history.
- Read and apply `.agents/skills/tasks/refactor/SKILL.md` whenever the user asks to
  refactor, restructure, simplify, extract, consolidate, split, or clean up
  existing code.
- After meaningful code edits and before claiming completion, always read and
  apply `.agents/skills/tasks/review/SKILL.md`. Apply it in read-only mode when
  the user requested review without implementation.
- Read and apply `.agents/skills/tasks/copy/SKILL.md` whenever writing, editing,
  or reviewing product, marketing, onboarding, interface, error, toast, prompt,
  documentation, pricing, or other user-facing language.
- Read and apply `.agents/skills/tasks/migrate/SKILL.md` whenever changing
  Supabase schema, persisted data shapes, indexes, functions, triggers, grants,
  storage policies, or RLS.
- Read and apply `.agents/skills/tasks/debug/SKILL.md` whenever diagnosing or
  fixing a bug, regression, failed check, unexpected behavior, or performance
  problem.
- Read and apply `.agents/skills/tasks/docs/SKILL.md` whenever changing
  documentation or shipped behavior involving setup, hosting, configuration,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hpbrn/creed](https://github.com/hpbrn/creed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->

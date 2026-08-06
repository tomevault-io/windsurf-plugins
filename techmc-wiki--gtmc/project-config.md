---
trigger: always_on
description: This file is the agent contract for the **GTMC website** repo. It complements `README.md` (human-facing) with the technical context, commands, and conventions an automated coding agent needs to make safe, useful changes.
---

# AGENTS.md

This file is the agent contract for the **GTMC website** repo. It complements `README.md` (human-facing) with the technical context, commands, and conventions an automated coding agent needs to make safe, useful changes.

> Agent-managed blocks at the bottom of this file (`<!-- gitnexus:* -->`, `<!-- BEGIN/END:nextjs-agent-rules -->`) are written by external tooling. **Do not hand-edit them.**

## Project Overview

This repo contains the public website for **Graduate Texts in Minecraft (GTMC)** — a community-driven online textbook on technical Minecraft. It serves articles (tutorials, mechanics explanations, source-code analyses) and a draft/review hub for contributors.

- **Framework**: Next.js 16 (App Router, Cache Components, Turbopack) on React 19
- **Language**: TypeScript 7.0.2 (strict mode)
- **Styling**: Tailwind CSS v4
- **Motion**: `motion` (Framer Motion successor)
- **Auth**: NextAuth v5 (GitHub provider) + Prisma adapter
- **Database**: Prisma 7 against Supabase Postgres
- **Content pipeline**: Markdown via remark/rehype, KaTeX math, Shiki code highlighting, gray-matter frontmatter
- **Editor**: CodeMirror 6 (markdown, autocomplete, merge view)
- **Schematics**: `schematic-renderer` + Three.js
- **Search**: MiniSearch
- **i18n**: `next-intl` with `en` and `zh` locales
- **Hosting**: Vercel (Speed Insights, Analytics, Blob)
- **Unified tooling**: pnpm 11 for package management, with Vite+ (`vp`) providing Oxlint, Oxfmt, and Vitest integration
- **Tests**: Vitest, Playwright, Lighthouse CI

The articles themselves live in a separate repo and are pulled in via a Git submodule at `articles/`, and the glossary CSV data is pulled in via a submodule at `glossary/`.

### Repository layout

```text
.
├── app/                    Next.js App Router (locale-scoped routes)
│   ├── [locale]/
│   │   ├── (public)/       Articles, public pages
│   │   ├── (private)/      Drafts, review hub, profile, admin
│   │   ├── (auth)/         GitHub sign-in flow
│   │   └── _homepage/      Hero card, foreground/background layers
│   └── api/                Route handlers
├── actions/                Server actions (drafts, reviews, profile, …)
├── components/ui/          tech-card, tech-button, corner-brackets, …
├── components/{articles,editor,glossary,layout,markdown,review,search,ui}/
├── lib/                    Article pipeline, auth, db, search, GitHub helpers
├── articles/               Article content (Git submodule)
├── glossary/               Glossary CSV data (Git submodule)
├── data/                   Generated manifest + rendered article content + glossary*.json
├── i18n/                   next-intl request config + routing
├── messages/               i18n catalogs (en.json, zh.json)
├── public/                 Static assets including generated gtmc-en.pdf and gtmc-zh.pdf
├── scripts/                Manifest, content, and PDF generators
├── proxy.ts                Auth + i18n middleware
├── schema.prisma           Database schema
└── DESIGN.md               Visual system reference
```

## Setup Commands

The project uses **pnpm v11** and runs on **Node 26** in CI. macOS, Linux, and Vercel build images are supported.

```bash
git clone https://github.com/techmc-wiki/gtmc.git
cd gtmc
vp install              # delegates to pinned pnpm; also runs postinstall
cp .env.example .env    # fill in GitHub OAuth, DATABASE_URL, etc.
pnpm dev                # http://localhost:3000
```

`pnpm install` triggers `scripts/postinstall.ts`, which:

1. Adds `.gitconfig` to the local Git config include path.
2. Initializes the `articles/` and `glossary/` submodules at their pinned commits if needed.
3. Generates the glossary manifest.
4. Runs `prisma generate` (with a placeholder `DATABASE_URL` if none is set, to allow client codegen offline), unless heavy postinstall steps are explicitly skipped.
5. Runs `tsx scripts/generate-article-manifest.ts`.
6. Runs `playwright install chromium` for the PDF generator and any browser tests, unless `GTMC_SKIP_PLAYWRIGHT=1` (or heavy postinstall is already skipped).

CI notes for the Build workflow:

- Content/PDF artifacts are cached by articles SHA + glossary SHA + generator/lib hashes. Exact cache hits set `GTMC_SKIP_CONTENT_BUILD=true`.
- Install uses `GTMC_SKIP_POSTINSTALL=1`; Prisma is generated in a dedicated step; Chromium is installed only on content-cache miss.
- Superseded runs on the same ref are cancelled via `concurrency`.

### Environment variables

`.env.example` lists the required keys. None are committed.

| Variable                                                   | Purpose                                                                          |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `DATABASE_URL`                                             | Pooled Postgres connection string (Supabase in production)                       |
| `DIRECT_URL`                                               | Direct Postgres connection used by Prisma migrations                             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [techmc-wiki/gtmc](https://github.com/techmc-wiki/gtmc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->

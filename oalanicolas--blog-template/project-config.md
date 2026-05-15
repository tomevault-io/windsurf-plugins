---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Status: Template Canônico

Este diretório (`apps/blog-template/`) é o **template canônico** do Sinkra Hub
para blogs automatizados de IA-indexação (GEO). Não é deployable por si só —
faltam `wrangler.toml`, `.env`, `database_id` (que vivem nos forks).

Para criar um fork por business × idioma, use **`scripts/scaffold-blog-business.sh`**:

```bash
scripts/scaffold-blog-business.sh aiox pt-BR
scripts/scaffold-blog-business.sh aiox en-US
scripts/scaffold-blog-business.sh allfluence pt-BR
```

Cada fork resultante é um Cloudflare Worker + D1 independente, com `SITE_LANG`
pré-preenchido e identidade de marca via `wrangler.toml`. Ver:

- **ADR-023** Multi-Site Per Language (1 Worker por business × idioma)
- **ADR-024** Canonical stack (Astro 6.2 + CF Workers + D1 + Drizzle)
- **ADR-025** Template + sync flow
- **ADR-026** robots.txt `Allow: /` intencional (citation > training-protection)
- **ADR-027** Conteúdo nativo (não tradução automática)

## Stack

- **Runtime:** Cloudflare Workers (edge SSR)
- **Framework:** Astro 6.2 (`output: 'server'`) com `@astrojs/cloudflare` adapter
- **Database:** Cloudflare D1 (SQLite na edge) via Drizzle ORM
- **UI:** Tailwind CSS 4 (CSS-first config via `@theme inline`) + Preact (única ilha: busca)
- **Linguagem:** TypeScript strict (`~/` = `src/`)
- **Package manager:** Bun (root `bun.lock` é o SOT; não usar `npm install`)
- **Idioma:** definido per-fork via `SITE_LANG` em `wrangler.toml` (BCP 47, ex: `pt-BR`, `en-US`, `es-ES`). Nunca hardcoded em código (per ADR-023).

## Tech-Debt Reconhecido

- **CSP API nativa** do Astro 6 ainda não habilitada — adicionar em sprint dedicado.
- **Built-in Fonts API** do Astro 6 ainda não usada — substitui `<link rel="preload">` manual (hoje Google Fonts via `<link>`).

## Comandos

```bash
bun run dev              # Astro dev server (com D1 local via platformProxy)
bun run build            # Astro build + post-build.mjs (.assetsignore)
bun run preview          # wrangler dev (runtime real do Worker localmente)
bun run deploy           # build + wrangler deploy (produção)
bun run typecheck        # astro check

bun run db:generate              # drizzle-kit generate (schema.ts -> SQL)
bun run db:migrate:local         # aplica migration inicial no D1 local
bun run db:migrate:remote        # aplica migration inicial no D1 remoto
bun run db:migrate:geo:local     # aplica migration GEO (hero_image, key_takeaways, faq, reading_time)
bun run db:migrate:geo:remote    # aplica migration GEO no D1 remoto
bun run db:migrate:rating:local  # aplica migration aggregate_rating (review/comparação)
bun run db:migrate:rating:remote # aplica migration aggregate_rating no D1 remoto
bun run db:migrate:geosquad:local  # aplica colunas de sinal da future content-geo squad
bun run db:migrate:geosquad:remote # aplica colunas de sinal no D1 remoto
bun run db:seed:local            # seed prod-safe: apenas categorias
bun run db:seed:remote           # seed remoto prod-safe: apenas categorias
bun run db:seed:dev:local        # demo article local
bun run db:seed:dev:staging      # demo article staging only; nunca produção
```

## Arquitetura

```
src/
  middleware.ts          # Auth gate: Bearer token em /api/* (exceto /api/search)
  db/schema.ts           # Drizzle schema: articles + categories
  db/client.ts           # createDb(d1) -> drizzle instance
  lib/                   # Lógica de negócio (validação, slug, SEO, pings, paths)
  lib/paths.ts           # Helper url() base-aware: emite /blog/... em todos os links internos
  pages/
    api/articles/        # CRUD REST (index.ts = list+create, [slug].ts = get+update+delete)
    api/publish/[slug].ts# Publicação: draft->published + IndexNow + Google ping
    api/search.ts        # Busca pública LIKE (sem auth)
    api/health.ts        # Health público (sem auth, ping D1)
    api/yt-transcript.ts # Extrai transcrição YouTube na edge (Bearer)
    api/taxonomy.ts      # Lista categorias + tags agregadas (Bearer)
    [slug].astro         # Página do artigo (SSR, JSON-LD, breadcrumbs, relacionados)
    categoria/[slug].astro
    index.astro          # Homepage com paginação + ilha de busca Preact
    sitemap.xml.ts       # Sitemap dinâmico (D1 query) + hreflang via SITE_ALTERNATES
    llms.txt.ts          # llmstxt.org standard — index LLM-friendly de articles publicados
    llms-full.txt.ts     # llmstxt.org full-content corpus — GEO distribution endpoint
    robots.txt.ts        # Allow: / total (decisão por ADR-026: citation > training-protection)
    [key].txt.ts         # IndexNow key verification (dinâmico, sem arquivo estático)
  layouts/Base.astro     # HTML shell: meta, OG/Twitter, JSON-LD Organization+WebSite, article:section/tag
  components/            # ArticleCard, Breadcrumb, Pagination, SearchIsland (Preact),
                         # AiShareButtons, KeyTakeaways, FaqBlock, TableOfContents
  lib/toc.ts             # Deriva sumário dos H2s + injeta ids para âncoras
  lib/structured-data.ts # Article / Breadcrumb / Organization / WebSite / FAQPage JSON-LD
.claude/skills/          # Skills do projeto (instruções carregadas sob demanda)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oalanicolas/blog-template](https://github.com/oalanicolas/blog-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->

---
trigger: always_on
description: **fazer.ai agents** é uma aplicação fullstack TypeScript (**Bun + Elysia + React 19 + Tailwind CSS v4**, Prisma/PostgreSQL, JWT, i18n, Biome) que roda **LangGraph TS** no backend para orquestrar agentes de atendimento (IA) sobre o **Chatwoot** fazer.ai. Construída sobre o template **bunfire** (cujos invariantes seguem documentados abaixo). Multi-tenant (`tenant_id` em tudo, isolamento por Prisma `$extends` + RLS), "um core, três transportes" (REST v1, MCP server, UI projetam sobre os mesmos serv
---

# CLAUDE.md

**fazer.ai agents** é uma aplicação fullstack TypeScript (**Bun + Elysia + React 19 + Tailwind CSS v4**, Prisma/PostgreSQL, JWT, i18n, Biome) que roda **LangGraph TS** no backend para orquestrar agentes de atendimento (IA) sobre o **Chatwoot** fazer.ai. Construída sobre o template **bunfire** (cujos invariantes seguem documentados abaixo). Multi-tenant (`tenant_id` em tudo, isolamento por Prisma `$extends` + RLS), "um core, três transportes" (REST v1, MCP server, UI projetam sobre os mesmos services), distribuição Free (open-source) vs Full.

Guia detalhado por subsistema vive em [`docs/`](docs/); as seções abaixo cobrem o que não cabe lá ou que você deve manter em memória de trabalho.


## Uma PR fecha uma issue, e issue que nenhuma PR fecha está errada

Toda PR sai com `Fixes #N`. O par é o teste de granularidade dos dois lados: se você não consegue apontar a issue que **esta** PR fecha, ou a PR está fazendo mais de uma coisa, ou a issue está pedindo mais de uma.

Quando é a issue que não cabe, **reescrevê-la é parte do trabalho e vem antes do código**: uma issue com três entregáveis vira três issues, cada uma com o seu `Fixes`. Trocar por `Refs` e seguir em frente é o que produz a issue que ninguém consegue fechar, aberta pela metade e sem conseguir dizer o que falta.

Isso é mais estrito do que o [`CONTRIBUTING.md`](CONTRIBUTING.md) pede de quem contribui de fora, e de propósito: lá a orientação é sobre como escrever uma issue, e uma PR sem issue continua bem-vinda; aqui vale para quem também escreve as issues.

Quando de fato não couber, isso aparece pela necessidade concreta e se resolve ali. Não há lista de exceções, e não é esquecimento: a lista é o caminho pelo qual um default vira sugestão.

## Subsystem docs

Each line says what the doc covers and when to open it. **The doc is the current text; this list is not** — when they disagree, the doc wins.

- [`docs/ui.md`](docs/ui.md): the operator console screen map, the shared client primitives, the tool-selection grant model, the i18n/extract gotchas, and the **app shell** (who renders `<Layout>`, `<PageContainer>`, the sidebar/menu contracts). Read before adding or changing any screen or client primitive.
- [`docs/auth.md`](docs/auth.md): **who may create an account here** — the `/setup` first-run invariant (advisory lock + count re-check, so it holds under concurrency), `SETUP_TOKEN_REQUIRED`, `SIGNUP_ENABLED`, the two domain allowlists, and the threat model behind auto-promotion to ADMIN. Read before touching any registration or promotion path.
- [`docs/google-oauth.md`](docs/google-oauth.md): Google Identity Services wiring, enable/disable steps. Read when adding social login or removing it from a derived project.
- [`docs/modals.md`](docs/modals.md): the `<Modal>` controller pattern, the always-render rule (enforced by lint), and a checklist for async modal flows. Read when adding any modal, especially one that fetches or mutates.
- [`docs/frontend-env-vars.md`](docs/frontend-env-vars.md): `BUN_PUBLIC_*` build-pipeline propagation (Bun `define`, `env.ts`, Dockerfile, CI workflow). Read when exposing a new env var to the browser.
- [`docs/cdn-r2-setup.md`](docs/cdn-r2-setup.md): Cloudflare R2 setup for CDN-served frontend assets (`BUN_PUBLIC_CDN_URL`), comparing the custom-domain and Worker approaches. Read when wiring the CDN for the first time.
- [`docs/routing.md`](docs/routing.md): **BrowserRouter + the `serve.routes` carve-out** — why deep-link refreshes work, why `/api/*` is not served as HTML, and the smoke test to re-run on **every Elysia upgrade**. Read before touching `src/app.ts` routing, the catch-all, or the build's `publicPath`.
- [`docs/csp.md`](docs/csp.md): the Content-Security-Policy `src/app.ts` builds (enforced in prod, Report-Only in dev), where the inline-script hashes come from, and what Google Sign-In changes (including the COOP relaxation). Read when adding an external script, style, font or API origin.
- [`docs/realtime.md`](docs/realtime.md): the WebSocket feature — four patterns, Bun pub/sub, the frontend hook contract, and two load-bearing Elysia 1.4.x gotchas that make a socket fail **silently**. Skim before any WS edit.
- [`docs/i18n.md`](docs/i18n.md): `t()` / `translate()` separation, magic comments for dynamic keys, biome lint plugins that catch missing or malformed translations. Read when adding user-facing text.
- [`docs/eden-treaty.md`](docs/eden-treaty.md): two non-negotiable rules for the Eden client, both of which fail **silently** when broken. Read before declaring any client-side type that consumes the treaty.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fazer-ai/agents](https://github.com/fazer-ai/agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

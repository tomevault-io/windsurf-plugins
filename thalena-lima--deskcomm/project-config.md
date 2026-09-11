---
trigger: always_on
description: > Contrato para **qualquer** agente de código (Codex, Cursor, Copilot, Amp, Claude Code).
---

# AGENTS.md — DeskcommCRM

> Contrato para **qualquer** agente de código (Codex, Cursor, Copilot, Amp, Claude Code).
> Este arquivo é o núcleo portável. A **doutrina completa e não-negociável vive em
> [`CLAUDE.md`](CLAUDE.md)** — leia-o antes de tocar em código. Aqui está o mínimo
> para não causar dano.

---

## Objetivo do projeto

Sistema operacional de vendas open source com agentes de IA nativos, multi-nicho,
WhatsApp como canal primário (via WAHA). Multi-tenant com RLS desde o dia 1, LGPD
nativa. Monetização = self-host em VPS, não assinatura. Posicionamento: [`VISION.md`](VISION.md).

**Consequência que muda como você trabalha:** o produto é distribuído como código.
Quem instala numa VPS **é** o usuário. Uma mudança que funciona na máquina do dev e
quebra no clone fresco é um bug de produto, não um detalhe de ambiente.

## Stack (CONFIRMADO em `package.json`)

Next.js 16 (App Router) · React 19 · TypeScript 6 estrito · Tailwind 4 ·
shadcn/ui · Supabase (Postgres + Auth + Realtime + Storage) · Upstash Redis ·
Vercel AI Gateway (`@ai-sdk/anthropic|openai|google`) · WAHA 2026.7.2 (engine NOWEB; sem bloqueio por tier) ·
Zod 4 · Vitest 4 · Playwright 1 · Sentry 10.

Só a **major**, de propósito: é onde o idioma muda, e é o que
`tests/unit/agents-md-versoes.test.ts` verifica contra o `package.json`. Declarar a minor
aqui fazia todo bump do Dependabot reprovar o `verify` (5 dos 8 pacotes) e não cobria nada
que a major já não cobrisse — issue #235. Para a versão exata, `package.json` é a fonte.

Runtime: **Node ≥22** (`.nvmrc` = 22; os quatro workflows fixam `node-version: 22` —
`ci` ×2, `perf`, `e2e`). Gerenciador: **pnpm 9.15.9** (`packageManager`).
Versão do produto: **não está escrita aqui, de propósito.** Esta linha afirmava `1.0.0` até a
v1.6.0 — seis minors de atraso, e nenhum teste a vigiava. Afirmação de versão envelhece a cada
release; comando não. A que está publicada agora:

```bash
git ls-remote --tags --refs origin 'refs/tags/v*' \
  | sed 's#.*refs/tags/v##' | awk '!/-/' | sort -V | tail -1   # awk, nao grep -v -- '-':
                                                                # em maquina com ugrep aquele nao roda
```

O `package.json` **não** é a fonte da versão do produto (segue em `0.1.0`, e é assim de
propósito). A fonte é a tag `v*` mais a seção do `CHANGELOG.md` — que é tela de produto, lida
pelo dono da VPS. Como o número é decidido: [`docs/doctrine/versionamento.md`](docs/doctrine/versionamento.md).

## Estrutura que importa

| Path | O quê |
|---|---|
| `app/api/v1/` | 166 route handlers REST (versionado por path) — 169 contando `app/api/**` |
| `app/api/internal/`, `app/api/mcp/`, `app/api/v1/cron/` | superfícies não-cookie (secret/bearer próprio) |
| `app/app/` | UI autenticada do tenant · `app/admin/` UI de plataforma |
| `app/actions/` | Server Actions (auth, onboarding, team, settings) |
| `lib/agent-engine/`, `lib/ai/` | runtime do agente, guardrails, RAG, dispatcher |
| `lib/api/wrappers.ts` | `ok()` / `fail()` — **use sempre**, não monte Response na mão |
| `lib/auth/require-role.ts` | `requireRole()` — guard canônico de RBAC |
| `lib/supabase/{browser,server,admin}.ts` | clients canônicos |
| `workers/` | workers de `event_log` + crons |
| `supabase/migrations/` | schema versionado · `supabase/baseline.sql` = o que o self-host aplica |
| `proxy.ts` | middleware do Next 16 (auth de borda, `X-Request-Id`) |

## Comandos (CONFIRMADO em `package.json`)

```bash
pnpm install          # deps (frozen-lockfile no CI)
pnpm dev              # dev server
pnpm build            # next build
pnpm lint             # eslint
pnpm typecheck        # tsc --noEmit (estrito)
pnpm test:unit        # vitest — EXCLUI tests/invariants e tests/e2e
pnpm test:db          # invariantes de banco + gate do baseline (PRECISA de Docker)
pnpm test:e2e         # Playwright (PRECISA de app rodando + banco semeado)
pnpm gov:verify       # typecheck + lint + test:unit  ← verificação única atual
```

⚠️ **`pnpm gov:verify` NÃO cobre tudo.** Ele omite `test:db` e `test:e2e`. Se sua
mudança toca schema, RLS ou UI, `gov:verify` verde **não** é prova — rode `pnpm test:db`
(exige Docker) e/ou `pnpm test:e2e` você mesmo. Ver [`docs/harness-audit.md`](docs/harness-audit.md).

**O que o CI cobre.** `.github/workflows/ci.yml`: `verify` = typecheck + lint + test:unit;
`invariants` = `pnpm test:db` (isolamento RLS + invariantes de governança contra Postgres
efêmero pg15). `.github/workflows/perf.yml`: `build-and-size` = `pnpm build`.
`.github/workflows/e2e.yml` roda as specs Playwright contra um Supabase local de verdade com
o `baseline.sql` aplicado — o mesmo banco que o self-hoster tem. **É check obrigatório desde
2026-08-08.** **Não há número aqui de propósito**: esta linha já afirmou uma contagem exata
de specs e "a única de fora", e as duas envelheceram — a suíte cresce toda semana e a lista de
exceções muda com ela. Quem fica de fora é o que a própria variável declara; leia, não confie:

```bash
git show origin/main:.github/workflows/e2e.yml | grep -A4 'FORA_DO_CI:'
```

O que continua verdade e é o que importa: `vps-fresh-onboarding` está entre elas (WAHA + Redis
+ Resend + Nuvemshop) e é a **P0** da doutrina de QA — ou seja, `e2e` verde não prova a jornada

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thalena-lima/deskcomm](https://github.com/thalena-lima/deskcomm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->

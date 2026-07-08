---
trigger: always_on
description: > **Leia inteiro antes de agir. Cada regra é operacional, não sugestão.**
---

# IOC ESG Municipal — CLAUDE.md

> **Leia inteiro antes de agir. Cada regra é operacional, não sugestão.**

---

## ⛔ PREMISSA ABSOLUTA E INEGOCIÁVEL — LEIA PRIMEIRO

```
O ÚNICO OBJETIVO ATUAL É:
Entregar o software funcionando para os usuários de Santa Catarina (SC)
e obter a aprovação do cliente final.

NÃO FAÇA NADA QUE NÃO SEJA NECESSÁRIO PARA O USUÁRIO FINAL DE SC USAR A PLATAFORMA.
```

**O que isso significa na prática:**

- **NÃO** implemente funcionalidades novas que não sejam demanda direta de uso do usuário de SC.
- **NÃO** evolua arquitetura, adicione camadas ou refatore se não for um bloqueador para o usuário de SC.
- **NÃO** expanda o escopo para outros estados, municípios fora de SC ou funcionalidades "nice to have".
- **NÃO** trabalhe em itens do backlog marcados como "Planejado" ou "Backlog" sem instrução explícita.
- **SIM** corrija bugs que impeçam o usuário de SC de usar a plataforma.
- **SIM** melhore estabilidade, dados e UX se for bloqueador para o uso real em SC.

**Só depois de aprovado o produto em SC pensaremos em expansão nacional, novas features ou evolução arquitetural.**

Documento de estado atual: `docs/ESTADO_ATUAL_SC.md` — leia sempre ao iniciar uma sessão.

---

## CONTEXTO DO PROJETO

**IOC ESG Municipal** = Plataforma SaaS B2G que ajuda prefeitos de Santa Catarina a investir FPM com impacto nos 17 ODS da ONU, com dados públicos reais, simulação de políticas e recomendação por IA.

**Mercado atual:** 295 municípios de Santa Catarina (FOCO EXCLUSIVO)

**Modelo:** Assinatura R$12k–200k/ano por município, 80%+ de margem

**Diferencial:** Dados públicos reais + simulação multi-agente + recomendação por IA

**Stack aprovada:**

- Backend: Node.js 20 + TypeScript strict + Express + Prisma ORM + PostgreSQL + Redis
- Frontend: React 18 + TypeScript + Vite + Tailwind CSS + Shadcn/ui + Recharts
- Testes: Vitest (unit/integration) + Playwright (e2e)
- Infra: Docker Compose (dev e prod)

---

## ESTADO OPERACIONAL — ONDE ESTÁ

Este arquivo é **instrução persistente**. Estado vivo (o que está pronto, o que falta, o que quebrou) **não** fica aqui.

| Fonte                                             | Conteúdo                                                            |
| ------------------------------------------------- | ------------------------------------------------------------------- |
| `docs/ESTADO_ATUAL_SC.md`                         | Projection humana oficial do estado atual (ler no início da sessão) |
| `logs/claude-sessions.jsonl`                      | Audit trail de sessões e eventos de runtime                         |
| `docs/architecture/CLAUDE_CODE_ADOCAO_IOC_ESG.md` | Baseline e matriz de aderência do projeto à arquitetura Claude Code |
| `.claude/GOTCHAS.md`                              | Armadilhas estáveis do domínio                                      |
| `docs/RUNBOOK_PRODUCAO.md`                        | Procedimentos operacionais de deploy, backup, SSL, alertas          |
| `docs/decisions/`                                 | ADRs — decisões arquiteturais                                       |

> **Nunca** escreva tabelas de status com ✅/⚠️/❌ neste arquivo. Elas mentem em ≤24h.

---

## INÍCIO DE SESSÃO — execute sempre

```bash
git log --oneline -5
git status --short | head -5
cat docs/ESTADO_ATUAL_SC.md
```

Reporte: **feito / em progresso / próximo passo exato**

---

## AMBIENTES — NÃO CONFUNDIR

| Ambiente          | Comando                                                                          | Composição                                                                           |
| ----------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **Dev (local)**   | `docker compose up --build`                                                      | `docker-compose.yml` — 4 containers: postgres, redis, backend (HMR), frontend (Vite) |
| **Prod (Docker)** | `docker compose -f docker-compose.prod.yml up -d`                                | Imagem multi-stage + nginx HTTP-only + postgres + redis                              |
| **Prod SSL**      | `docker compose -f docker-compose.prod.yml -f docker-compose.prod.ssl.yml up -d` | Override com certificados Let's Encrypt                                              |

**Regra operacional:** toda alteração em `backend/`, `shared/types/`, `prisma/` ou rotas exige rebuild da imagem antes de ser considerada concluída:

```bash
docker build -t ioc-esg-municipal:$(git rev-parse --short HEAD) . && echo "prod build OK"
```

Reporte explícito na mensagem de conclusão: `docker build: OK | tsc: OK | tests: OK`.

---

## COMANDOS DO PROJETO

### Dev (Docker)

```bash
docker compose up --build        # sobe os 4 containers com HMR
docker compose down
docker compose logs -f backend
pnpm db:seed                     # seed 295 municípios SC
pnpm db:migrate                  # prisma migrate dev
pnpm test                        # unit + integration
pnpm test:e2e                    # playwright
pnpm lint && pnpm format
```

### Produção

```bash
# Build da imagem multi-stage
docker build -t ioc-esg-municipal:latest .

# Subir stack de produção HTTP-only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Joubertjr/ioc-esg-municipal](https://github.com/Joubertjr/ioc-esg-municipal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->

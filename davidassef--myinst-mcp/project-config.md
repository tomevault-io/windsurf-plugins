---
trigger: always_on
description: > Este arquivo é destinado a agentes de código. Leia-o antes de modificar qualquer parte do projeto.
---

# AGENTS.md — MyInst

> Este arquivo é destinado a agentes de código. Leia-o antes de modificar qualquer parte do projeto.
> Idioma do projeto: **português (pt-BR)** para documentação, comentários e nomes de negócio; inglês para APIs/frameworks.

---

## Visão geral do projeto

O **MyInst** é um vault open source para armazenar, versionar e sincronizar contexto agentic entre projetos, workspaces, dispositivos e clientes MCP.

Ele centraliza `skills`, `instructions`, `agents`, `hooks`, `memory`, `snippets` e configurações de clientes em um backend próprio, com interface web, API, CLI e um MCP server local.

Componentes principais:

| Componente | Pacote / Pasta | Papel |
|------------|----------------|-------|
| Frontend | `frontend/` | Painel web (React + Vite) para gerenciar workspaces, projetos, conteúdo e API keys |
| Backend | `backend/` | API Fastify com auth, busca, sync, versionamento e persistência |
| CLI | `packages/cli/` | CLI publicável no npm (`myinst`) para login/list/pull/push fora do fluxo MCP |
| MCP Server | `packages/mcp-server/` | Servidor MCP local publicável (`myinst-mcp`) que conecta clientes ao vault |
| Shared | `packages/shared/` | Schemas Zod, tipos TypeScript e constantes compartilhados |

Repositório: `git@github.com:davidassef/MyInst-mcp.git`
Licença: **AGPL-3.0**

---

## Stack e pré-requisitos

- **Linguagem:** TypeScript 5.8+
- **Runtime:** Node.js **>= 22.0.0**
- **Package manager:** pnpm **10.28.0** (configurado via `packageManager` no `package.json`; `corepack` ativado nos Dockerfiles)
- **Monorepo:** pnpm workspaces + Turborepo 2.5+
- **Backend:** Fastify 5.3, Drizzle ORM 0.44, PostgreSQL 16+, Zod, bcrypt, nanoid, diff
- **Frontend:** React 19, React Router DOM 7, Vite 6, Tailwind CSS 4, lucide-react
- **MCP:** `@modelcontextprotocol/sdk`
- **Testes:** Vitest 3+
- **Containeres:** Docker + Docker Compose

---

## Arquivos de configuração principais

### Raiz

- `package.json` — scripts do monorepo, dependências comuns, `packageManager`, `engines`
- `pnpm-workspace.yaml` — define `frontend`, `backend` e `packages/*`
- `turbo.json` — pipeline de `build`, `dev`, `test`, `lint`
- `tsconfig.base.json` — configuração base TypeScript para todo o monorepo
- `.env.example` — variáveis de ambiente obrigatórias/opcionais
- `docker-compose.yml` — stack local de desenvolvimento (API + Postgres)
- `Dockerfile` — build de produção da API (compila `shared` e `backend`)
- `Dockerfile.web` — build do frontend servido por nginx

### Backend

- `backend/package.json` — scripts e dependências
- `backend/tsconfig.json` — extende `tsconfig.base.json`
- `backend/drizzle.config.ts` — configuração do Drizzle Kit
- `backend/vitest.config.ts` — configuração dos testes

### Frontend

- `frontend/package.json`
- `frontend/tsconfig.json`
- `frontend/vite.config.ts` — proxy `/api` para `localhost:3000` quando `VITE_MYINST_API_BASE` está vazio
- `frontend/vercel.json` — configuração de deploy na Vercel (Root Directory = `frontend`)

### Pacotes publicáveis

- `packages/shared/package.json` — `@myinst/shared`
- `packages/mcp-server/package.json` — `@myinst/mcp-server`, binário `myinst-mcp`
- `packages/cli/package.json` — `@myinst/cli`, binário `myinst`

---

## Comandos de build, dev e teste

Todos os comandos partem da raiz do monorepo.

```bash
# Instalação
pnpm install

# Ambiente local
cp .env.example .env
pnpm db:push      # cria/atualiza as tabelas no Postgres local
pnpm dev          # sobe backend (localhost:3000) e frontend (localhost:5173)

# Comandos individuais
pnpm dev:backend   # turbo dev com filtro no backend
pnpm dev:frontend  # turbo dev com filtro no frontend

# Build
pnpm build         # turbo build (depende de ^build)

# Validação
pnpm lint          # turbo lint = geralmente tsc --noEmit em cada pacote
pnpm test          # turbo test
pnpm validate      # lint + build + test

# Docker / deploy local
pnpm compose:check      # valida todos os docker-compose.yml do projeto
pnpm prod:preflight     # simula produção localmente (build, sobe Postgres, schema, API, smoke)
pnpm smoke              # executa smoke test contra a API (MYINST_SMOKE_BASE_URL)

# Banco de dados
pnpm db:push            # drizzle-kit push no backend
pnpm db:migrate         # drizzle-kit migrate no backend
pnpm db:studio          # drizzle-kit studio
pnpm db:deploy:schema   # aplica schema via container Docker (usado em deploy)
pnpm db:backup          # backup do banco
pnpm db:restore <arquivo.sql>  # restore (exige MYINST_CONFIRM_RESTORE=CONFIRMO_RESTORE)
```

### Comandos por pacote

```bash
pnpm --filter @myinst/backend lint
pnpm --filter @myinst/backend test
pnpm --filter @myinst/frontend build
pnpm --filter @myinst/mcp-server test
pnpm --filter @myinst/cli test
```

---

## Organização do código

### Backend (`backend/src/`)

```
backend/src/
├── index.ts            # entrypoint: carrega env, cria app, seed e listen
├── app.ts              # factory do Fastify, plugins (cors, helmet, jwt, rate-limit) e rotas
├── env.ts              # validação de variáveis de ambiente
├── routes/             # handlers da API
│   ├── auth.ts
│   ├── oauth.ts
│   ├── workspaces.ts
│   ├── projects.ts
│   ├── content.ts
│   ├── sync.ts
│   ├── tags.ts
│   ├── search.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidassef/MyInst-mcp](https://github.com/davidassef/MyInst-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->

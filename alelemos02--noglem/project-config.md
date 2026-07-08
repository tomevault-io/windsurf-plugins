---
trigger: always_on
description: Instrucoes universais para qualquer agente de IA trabalhando neste projeto.
---

# AGENTS.md — JulIA Engineering Platform

Instrucoes universais para qualquer agente de IA trabalhando neste projeto.
Valido para Claude, GPT, Gemini, Codex ou qualquer outro LLM.

---

## O que e este projeto

**JulIA** (Julia + IA) e uma plataforma centralizada de ferramentas de engenharia com IA.
Oferece: traducao de documentos, extracao de tabelas PDF, conversao PDF→Word, RAG (base de conhecimento), pareceres tecnicos e extracao de instrumentos de P&ID.

**Tom obrigatorio:** tecnico, direto, preciso. Nunca marketing. Nunca linguagem casual ou infantilizada.

---

## Arquitetura

```
Browser
  └── Next.js 16 App Router (frontend + API Routes como proxy seguro)
        ├── Backend Central (FastAPI, porta 8000)   ← ferramentas simples
        ├── PATEC Microservice (FastAPI + PostgreSQL + Celery + Redis)
        └── Conhecimento/RAG Microservice (FastAPI + PostgreSQL/pgvector + FlashRank)
```

### Regra de seguranca — nunca violar

Toda requisicao do Next.js para os backends deve incluir:
- `X-Internal-API-Key` — chave compartilhada validada no backend
- `X-User-Id` — Clerk User ID do usuario autenticado

O helper `buildBackendAuthHeaders()` em `src/lib/backend.ts` monta essas headers.
**Nunca chame os backends diretamente do browser. Sempre via API Route do Next.js.**

Rotas protegidas: todo `/dashboard/*` requer autenticacao Clerk (middleware.ts).
Rotas publicas: `/`, `/sign-in`, `/sign-up`, webhooks.

---

## Estrutura de Diretorios

```
enghub-v2/
├── src/
│   ├── app/
│   │   ├── (auth)/                  # Paginas de autenticacao Clerk
│   │   ├── dashboard/               # Uma pasta por ferramenta
│   │   │   ├── translate/
│   │   │   ├── pdf-extractor/
│   │   │   ├── pdf-converter/
│   │   │   ├── pid-extractor/
│   │   │   ├── rag/
│   │   │   └── parecer-tecnico/
│   │   ├── api/                     # API Routes (proxies seguros para os backends)
│   │   └── globals.css              # Tokens CSS do design system
│   ├── components/
│   │   ├── ui/                      # Design system: Button, Card, Badge, Input, Logo, Skeleton
│   │   ├── layout/                  # Shell, Header, Sidebar
│   │   └── parecer-tecnico/         # Componentes especificos do PATEC
│   ├── lib/
│   │   ├── backend.ts               # buildBackendAuthHeaders()
│   │   ├── tools-registry.ts        # Registro central de todas as ferramentas
│   │   └── utils.ts                 # cn() e helpers gerais
│   └── design-system/               # Tokens e referencia visual
├── backend/                         # FastAPI central
│   └── app/
│       ├── routers/                 # translate, pdf, pid, email
│       └── services/                # gemini, pdf_extract, pdf_convert, pid_extract
├── services/
│   ├── patec-backend/               # Microservico pareceres tecnicos
│   └── conhecimento-backend/        # Microservico RAG/conhecimento
├── AGENTS.md                        # Este arquivo — instrucoes universais
└── CLAUDE.md                        # Instrucoes especificas para Claude Code
```

---

## Tools Registry

`src/lib/tools-registry.ts` e a **fonte unica de verdade** para todas as ferramentas.
Usado pela sidebar, dashboard grid e status badges. Qualquer alteracao de metadados de ferramenta passa por aqui.

Campos obrigatorios: `id`, `name`, `description`, `icon`, `href`, `category`, `status`
- `status`: `'live'` | `'beta'` | `'coming_soon'`
- `category`: `'documentacao'` | `'conhecimento'` | `'analise'` | `'instrumentacao'`

---

## Como Alterar uma Ferramenta Existente

**Regra zero: leia os arquivos envolvidos antes de qualquer mudanca.**

### 1. So UI (layout, texto, estilo)
- Edite `src/app/dashboard/<tool>/page.tsx` e componentes em `src/components/<tool>/`
- Siga o design system — tokens CSS, tipografia, border radius
- Nao toque API Route nem backend

### 2. Logica de frontend (estado, interacao)
- Edite a pagina ou componentes filhos
- Verifique se o endpoint existente ja retorna os dados necessarios antes de criar um novo
- Prefira adicionar parametros opcionais ao endpoint existente

### 3. API Route (proxy Next.js)
- Arquivo: `src/app/api/<tool>/route.ts`
- Nunca remova a validacao de auth (`X-Internal-API-Key`, `X-User-Id`)
- Normalize erros — nunca exponha stack traces do backend ao browser
- Se mudar o contrato (body, params), atualize frontend e backend juntos

### 4. Backend Central
- Router: `backend/app/routers/<tool>.py`
- Service: `backend/app/services/<tool>_service.py`
- Mantenha `require_internal_api_key` em todos os endpoints
- Se mudar schema de resposta, atualize os tipos TypeScript no frontend

### 5. PATEC Microservice
- Endpoints: `services/patec-backend/app/api/v1/endpoints/`
- Mudancas no banco: sempre crie migration via `alembic revision --autogenerate`
- Nunca altere o banco diretamente

### 6. RAG Microservice
- Services: `services/rag-backend/app/services/`
- Mudancas no modelo de embedding afetam documentos ja indexados — documente o impacto

### 7. Metadados de ferramenta
- Edite apenas `src/lib/tools-registry.ts`

### Regras gerais
- Altere apenas o que foi pedido. Nao refatore codigo adjacente.
- Nao adicione tratamento de erro para cenarios que nao podem ocorrer.
- Nao crie helpers ou abstracoes para uso unico.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alelemos02/Noglem](https://github.com/alelemos02/Noglem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->

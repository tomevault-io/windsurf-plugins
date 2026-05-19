---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Visão Geral

**IntellexIA** é uma plataforma de automação jurídica com IA, focada em **direito trabalhista e previdenciário** (especialmente casos de FAP — Fator Acidentário de Prevenção). O sistema gerencia processos judiciais, analisa documentos, gera petições e oferece uma base de conhecimento consultável via agentes de IA.

---

## Stack Tecnológico

| Camada     | Tecnologia                                     |
| ---------- | ---------------------------------------------- |
| Backend    | Python 3.11–3.13 + Flask 3.1                   |
| ORM        | SQLAlchemy via Flask-SQLAlchemy                |
| LLM        | OpenAI (GPT-4o-mini, GPT-5-mini) via LangChain |
| Vector DB  | Qdrant (busca semântica) + FAISS (local)       |
| Full-text  | Meilisearch                                    |
| Documentos | Docling, PyMuPDF, pdfplumber, python-docx      |
| DB Dev     | SQLite (`instance/intellexia.db`)              |
| DB Prod    | MySQL 8.0 (via `pymysql`)                      |
| Infra      | Docker Compose (MySQL + Qdrant + Meilisearch)  |
| Frontend   | Jinja2 + AdminLTE 4 + Bootstrap 5              |
| Deps       | `uv` (não use `pip` diretamente)               |

---

## Comandos de Desenvolvimento

```bash
# Subir infra (MySQL, Qdrant, Meilisearch)
docker compose -f docker/docker-compose.yml up -d

# Instalar dependências
uv sync

# Rodar aplicação (dev — SQLite, debug ativo, cria tabelas automaticamente)
uv run python main.py
# ou
uv run flask run

# Produção (MySQL + Gunicorn)
uv run gunicorn -w 4 -b 127.0.0.1:8000 wsgi:app
```

### Testes

Não há framework de testes configurado. Os arquivos em `tests/` e `scripts/tests/` são **scripts executáveis** que importam `main.app` e usam `app.test_client()` / `app.app_context()`. Rode-os individualmente:

```bash
uv run python tests/test_dashboard.py
uv run python scripts/tests/test_document_extractor.py --knowledge-id 123
```

### Migrations

**Não há Alembic.** Cada migração é um script Python isolado em `database/` (ex.: `add_fap_reason_column.py`, `add_benefits_table.py`). Rode manualmente com `uv run python database/<script>.py`. Para recriar do zero: `uv run python database/recreate_database.py` (APAGA TUDO). Novas migrations seguem este padrão de script standalone.

---

## Estrutura de Diretórios

```
intellexia/
├── app/
│   ├── agents/                  # Agentes de IA
│   │   ├── core/                # FileAgent (upload OpenAI)
│   │   ├── document_processing/ # Extração e análise de documentos
│   │   ├── knowledge_base/      # RAG: roteamento, query, ingestão
│   │   ├── legal_drafting/      # Geração de petições
│   │   └── fap/                 # Classificador/gerador de seções FAP
│   ├── blueprints/              # Rotas Flask (uma pasta por módulo)
│   ├── services/                # Camada de serviços (orquestração)
│   ├── middlewares.py           # Auth + context processors
│   ├── models.py                # Modelos SQLAlchemy (~2100 linhas)
│   ├── prompts/                 # Prompts de agentes
│   └── utils/                   # timezone.py (SP_TZ), document_utils.py
├── database/                    # Scripts de migration (não-Alembic)
├── docker/                      # docker-compose.yml + configs
├── docs/                        # Documentação funcional (40+ .md)
├── scripts/                     # Scripts utilitários + scripts/tests/
├── templates/                   # Jinja2 (AdminLTE)
├── static/                      # Assets
├── tests/                       # Scripts de teste standalone
└── main.py                      # Entry point — define `app` Flask
```

**Código legado (evitar modificar; consulte antes de reutilizar):**
- `app/routes.py` — antiga, substituída por blueprints. Mantém apenas `/api/health` e rota de teste.
- `app/routes_backup.py`, `old/`, `agent_document_generator.py` na raiz — arquivos históricos.

---

## Arquitetura da Aplicação

### Entry Point e Configuração

`main.py` carrega `.env` manualmente (não usa `python-dotenv` em runtime), define timezone global `America/Sao_Paulo`, escolhe SQLite/MySQL via `ENVIRONMENT`, e registra **todos os blueprints** e middlewares. Instâncias de teste/scripts que precisam do contexto Flask devem fazer `from main import app` e usar `with app.app_context():`.

### Blueprints Registrados

`auth`, `dashboard`, `cases`, `clients`, `lawyers`, `courts`, `benefits`, `documents`, `petitions`, `assistant`, `tools`, `settings`, `knowledge_base`, `admin_users`, `process_panel`, `disputes_center`, `case_comments`, `fap_reasons`. Cada um em `app/blueprints/<nome>.py`, expondo `<nome>_bp`.

### Multi-Tenancy (CRÍTICO)

O sistema é multi-tenant por **escritório de advocacia** (`LawFirm`). Quase todas as tabelas de negócio carregam `law_firm_id`. Toda query de listagem DEVE filtrar por `law_firm_id`:

```python
law_firm_id = session.get('law_firm_id')  # ou get_current_law_firm_id()
Case.query.filter_by(law_firm_id=law_firm_id)...
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thiagoscheidt/intellexia](https://github.com/thiagoscheidt/intellexia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

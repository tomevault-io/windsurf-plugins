---
trigger: always_on
description: Memoria colectiva compartida para equipos multi-agente. PostgreSQL + pgvector + Apache AGE + Jina v4 + GLiNER + MCP.
---

# EcoDB — Project CLAUDE.md

Memoria colectiva compartida para equipos multi-agente. PostgreSQL + pgvector + Apache AGE + Jina v4 + GLiNER + MCP.

## Versiones actuales

- API: `0.23.0` (imagen Docker) / API_VERSION `0.8.1`
- Schema: `5.0.1`
- MCP: `1.6.0`
- Embeddings: `0.2.5`
- NER: `1.0.0`
- Postgres: `1.0.0` (PG16 + pgvector + AGE 1.5.0)
- Release pública: `v0.8.5`

## Arquitectura — 6 servicios Docker

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   MCP :8091 │────▶│  API :8080  │────▶│ Postgres    │
│  (proxy)    │     │  (FastAPI)  │     │ :5435→5432  │
└─────────────┘     └──┬───┬──┬───┘     │ pgvector    │
                       │   │  │         │ AGE graph   │
              ┌────────┘   │  └───────┐ │ pg_trgm     │
              ▼            ▼          ▼ └─────────────┘
     ┌────────────┐  ┌──────────┐  ┌──────┐
     │ Embeddings │  │   NER    │  │ LLM  │
     │ Jina v4    │  │ GLiNER   │  │ opt. │
     │ GPU/CUDA   │  │ CPU      │  │ CPU  │
     │ (interno)  │  │ :8092→   │  └──────┘
     └────────────┘  │    8091  │
                     └──────────┘
```

- **postgres** — PG16, imagen custom con AGE compilado. Data en named volume `ecodb_data`. Puerto host `127.0.0.1:5435`.
- **api** — FastAPI + uvicorn. Motor GAMR (10 etapas), auth JWT, CRUD. Puerto `8080`.
- **embeddings** — Jina v4 512-dim, INT8 GPU. Solo accesible internamente (sin puerto host). HF cache bind-mount `:ro`.
- **mcp** — Proxy MCP→API. SSE transport. Puerto `8091`. Sin lógica de negocio.
- **ner** — GLiNER NER para extracción de entidades. CPU-only. Puerto host `8092`, interno `8091`.
- **llm** (opcional, profile `with-llm`) — llama.cpp + Qwen 2.5 3B. Para clasificación, HyDE.
- **worker** (opcional, profile `with-ingestion`) — Ingesta de documentos (PDF/DOCX/audio).

## Estructura del repo

```
EcoDB/
├── api/                    # FastAPI backend — TODA la lógica de negocio
│   ├── main.py             # App factory, routers, middleware
│   ├── settings.py         # Config central (env vars, feature flags, GAMR weights)
│   ├── search.py           # Motor GAMR (62K, archivo más grande)
│   ├── memories.py         # CRUD memorias (45K)
│   ├── graph.py            # Grafo AGE — Cypher queries (36K)
│   ├── admin.py            # Endpoints admin (34K)
│   ├── auth.py             # JWT + API keys
│   ├── permissions.py      # Cascada de permisos (workspace→project)
│   ├── worker.py           # Pipeline ingesta documentos (23K)
│   ├── gliner_service.py   # NER client + entity dictionary
│   ├── embeddings_client.py # Client httpx → embeddings service
│   ├── reranker.py         # Cross-encoder (Etapa 10 GAMR)
│   ├── bootstrap_first_apikey.py  # Script primer arranque
│   ├── Dockerfile
│   ├── requirements.txt
│   └── tests/              # pytest — 17 archivos de test
├── mcp/
│   ├── server.py           # 32 tools MCP, proxy puro (61K)
│   ├── Dockerfile
│   └── requirements.txt
├── embeddings/
│   ├── server.py           # Jina v4 embedding server (21K)
│   └── Dockerfile
├── ner/
│   ├── server.py           # GLiNER NER server (1.4K)
│   └── Dockerfile
├── sql/
│   ├── init.sql            # Schema completo (38K) — se ejecuta en primer boot
│   └── migrate_*.sql       # Migraciones incrementales
├── docker/
│   └── Dockerfile.postgres # PG16 + pgvector + AGE custom build
├── scripts/
│   ├── setup.sh            # Bootstrap: genera .env, verifica deps
│   ├── backup.sh / restore.sh
│   └── seed_predicates.py  # Semilla del grafo de predicados
├── eval/                   # Benchmarks (LoCoMo, golden set, latencia)
├── docs/
│   ├── architecture/       # Briefs de diseño (governance, ingestion, intelligence, product)
│   └── plans/              # Planes de construcción por sesión
├── docker-compose.yml      # Compose principal
├── docker-compose.seed.yml # Dataset demo
├── .env.example
├── CHANGELOG.md
└── README.md
```

## Schema de base de datos (init.sql)

Tablas principales:
- `users` — con `is_super` (único) e `is_ceo` (mutuamente excluyentes)
- `user_emails` — tabla puente, email como PK (unicidad global)
- `organizations` — empresa cliente, 1:1 con CEO
- `workspaces` — departamento dentro de org
- `projects` — dentro de workspace, `is_common` para proyectos compartidos
- `workspace_leads`, `project_leads`, `project_members` — permisos por rol
- `teams`, `team_members`, `team_resources` — equipos ad-hoc cross-workspace
- `memories` — con `embedding vector(512)`, `visibility`, `type`, `tags TEXT[]`, soft-delete
- `agent_identity` — fragmentos ordenados por `(agent_id, version, fragment_idx)`
- `memory_type_config` — pesos base y decay por tipo
- `entity_dictionary` — diccionario curado para NER
- `entity_links` — links entidad↔memoria
- `documents`, `document_chunks` — ingesta documental
- `schema_version` — versionado de schema

Grafo: `ecodb_graph` en Apache AGE (Cypher). Sync automático via triggers SQL→AGE.

## Motor GAMR — 10 etapas

1. Clasificación query_type (factual/historical/analytical/contextual)
2. Filtro permisos cascada
3. Búsqueda semántica coseno (pgvector HNSW)
4. BM25 lexical (pg_trgm)
5. Expansión por grafo (AGE Cypher)
6. Resolución de fuentes
7. Coherencia temporal (freshness scoring)
8. Detección de contradicciones

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [josortmel/EcoDB](https://github.com/josortmel/EcoDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

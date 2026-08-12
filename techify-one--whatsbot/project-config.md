---
trigger: always_on
description: Bot de WhatsApp com IA para usuários finais, distribuído como EXE Windows.
---

# WhatsBot

Bot de WhatsApp com IA para usuários finais, distribuído como EXE Windows.

## Stack

- **Python 3.11+** — linguagem principal
- **SQLAlchemy 2.0 Core + Alembic** — camada de dados portável (Core, sem ORM declarativo)
- **SQLite** — banco default (WAL mode, driver `sqlite3` da stdlib)
- **PostgreSQL** — backend opcional via `psycopg[binary]`, configurável pela tela Settings → Banco
- **GOWA** (go-whatsapp-web-multidevice v8.8.0) — bridge WhatsApp via REST, roda como subprocess
- **Proxy LLM da Techify** (`https://llm.techify.one/api/v1`) — provider de LLM, API **compatível com OpenRouter/OpenAI**. Substituiu o OpenRouter direto: a chave é provisionada pelo wizard de 1ª execução e o crédito/recarga é gerido pela Techify. O base URL é configurável via env `LLM_API_BASE_URL`. A chave continua sendo persistida na config key `openrouter_api_key` (nome legado mantido por compatibilidade)
- **AGNO** (`agno` 2.x) — framework de agentes usado como **motor de LLM** do agente. O loop de raciocínio + tool calling roda via `agno.agent.Agent`, apontado ao proxy Techify pelo model `OpenAILike`. Encapsulado em [agent/agno_engine.py](agent/agno_engine.py); o `AgentHandler` delega a ele preservando todos os hooks de plugin (filters/events), usage e execution tracking. Transcrição de áudio/descrição de imagem continuam em chamadas diretas ao cliente OpenAI (não são agênticas)
- **FastAPI + uvicorn** — backend web (REST API + WebSocket)
- **Preact + HTM + Tailwind CSS** — frontend web (sem build step, vendorizado local)
- **PyInstaller** — empacotamento como EXE

## Arquitetura

```
main.py              → entry point, inicia uvicorn + abre browser
server/app.py        → FastAPI app (endpoints REST, WebSocket, webhook, background tasks)
gowa/manager.py      → lifecycle do subprocess GOWA (start/stop/watchdog)
gowa/client.py       → HTTP client para REST API do GOWA (localhost:3000)
agent/handler.py     → orquestra o processamento de mensagens (system prompt, filters/events, usage, save); delega o loop de LLM ao motor AGNO
agent/agno_engine.py → motor AGNO: monta OpenAILike + Agent único, envolve cada tool em agno Function (filters/events preservados), extrai reply/usage
agent/memory.py      → ContactMemory e TagRegistry (leitura/escrita no SQLite via repos)
agent/group_mentions.py → resolução de @menções em grupos (número ↔ nome, lista de membros, @todos)
agent/tools/         → tools core do LLM (uma tool por arquivo, agregadas em CORE_TOOLS)
config/settings.py   → load/save config + constantes do provider/Techify (LLM_API_BASE_URL, TECHIFY_*)
server/avatars.py    → cache de fotos de perfil em disco (statics/avatars/<phone>.jpg) + broadcast avatar_updated
server/balance_monitor.py → consulta saldo de crédito do proxy (/credits) e emite low_balance via WS
db/                  → módulo de banco de dados (SQLAlchemy 2.0 Core)
  engine.py          → factory do Engine, URL resolution (env > arquivo > sqlite default), PRAGMAs SQLite
  tables.py          → MetaData + 11 Table objects (Core, sem mapper/Session)
  upsert.py          → helper dialect-agnóstico (INSERT ... ON CONFLICT)
  connection.py      → init_db(): cria engine + roda Alembic upgrade
  migration_postgres.py → migra dados SQLite → Postgres (usado pelo endpoint admin)
  migrate_json.py    → migração one-time de JSON legado → banco
  alembic/           → migrations Alembic (env.py + versions/)
  repositories/      → data access layer (um arquivo por domínio)
    config_repo.py   → get_all(), get(), set(), set_many(), delete_prefix()
    contact_repo.py  → get_or_create(), update(), list_contacts(), get_full_contact()
    message_repo.py  → add(), get_all(), get_context(), get_last(), delete_all()
    usage_repo.py    → add(), global_summary(), by_contact(), detail()
    tag_repo.py      → get_all(), create(), update(), delete(), set_contact_tags()
    plugin_repo.py   → list_all(), upsert(), set_enabled(), applied_migrations()
plugins/             → sistema de plugins (core, não confundir com storages/plugins)
  loader.py          → PluginRegistry, descoberta + importlib + bootstrap
  manifest.py        → parser plugin.yaml + validação semver
  migrator.py        → runner SQL com prefixo plugin_<id>_ obrigatório
  context.py         → ToolContext, PromptContext (passados aos plugins)
  restart.py         → schedule_restart() — touch sentinela + os._exit
assets/              → recursos não-código (templates copiados em runtime)
  plugin_examples/   → plugins de referência (copiados pra storages/plugins/ no 1º boot)
storages/plugins/    → user-writable, ignorado por .gitignore (preservado em updates)
web/index.html       → entry point do frontend (HTML + import map)
web/static/js/       → componentes Preact + HTM (sem build step)
web/static/vendor/   → libs JS vendorizadas (preact, htm, tailwind)
bin/gowa.exe         → binário GOWA pré-compilado (não editar)
```

## Comandos

Escolha o launcher pelo ambiente onde está rodando:

| Ambiente | Comando | Modo | Hot-reload | Quando usar |
|---|---|---|---|---|
| Linux dev nativo | `./linux_start.sh` | Python local + uvicorn `--reload` | Sim (core + plugins) | Dia-a-dia de desenvolvimento — edita `.py` e o worker reinicia sozinho |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Techify-one/whatsbot](https://github.com/Techify-one/whatsbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

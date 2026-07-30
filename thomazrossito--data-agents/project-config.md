---
trigger: always_on
description: Sistema multi-agente construído sobre o **Claude Agent SDK** da Anthropic com integração
---

# Data Agents — Guia para Claude Code

Sistema multi-agente construído sobre o **Claude Agent SDK** da Anthropic com integração
nativa via MCP ao **Databricks** e **Microsoft Fabric**. Orquestra 14 agentes especialistas
em Engenharia, Qualidade, Governança, Análise de Dados e Web Semântica.

---

## Como Rodar

```bash
# Setup (uma vez)
pip install -e ".[dev,ui,monitoring]"
cp .env.example .env   # preencher credenciais

# Execução
python main.py                        # CLI interativo
python main.py "liste tabelas silver" # single-query
./start.sh                            # Web UI (Chat + Monitoring)
./start.sh --chat-only                # Só o chat Chainlit (porta 8503)

# Qualidade
make test        # pytest com cobertura (mínimo 80%)
make lint        # ruff check
make format      # ruff format
make type-check  # mypy
make health-databricks
make health-fabric
```

---

## Arquitetura de Alto Nível

```
Usuário → main.py / ui/chainlit_app.py
  └─► Supervisor (claude-sonnet-4-6, sem MCP direto)
        ├─► business-analyst      [T3] — intake de requisitos, /brief
        ├─► sql-expert            [T1] — SQL, schemas, catálogos
        ├─► spark-expert          [T1] — PySpark, DLT, Delta Lake
        ├─► python-expert        [T1] — Python puro: pacotes, APIs, CLIs, testes
        ├─► pipeline-architect   [T1] — ETL/ELT cross-platform
        ├─► migration-expert     [T1] — Migração SQL Server/PostgreSQL → Databricks/Fabric
        ├─► dbt-expert         [T2] — dbt Core: models, testes, snapshots
        ├─► data-quality-steward [T2] — validação, profiling, SLA
        ├─► governance-auditor   [T2] — auditoria, LGPD, linhagem
        ├─► semantic-modeler      [T2] — modelos semânticos, DAX, Genie
        ├─► catalog-intelligence  [T2] — comentários AI, Data Maturity Score, valor de negócio (/catalog)
        ├─► ontology-engineer     [T2] — ontologias OWL 2, import/export OneLake, rdflib, triples → Delta (/ontology)
        ├─► business-monitor      [T2] — Q&A interativo sobre alertas (daemon em `scripts/monitor_daemon.py`)
        └─► geral                [T0] — perguntas conceituais, zero MCP (Haiku)
```

**Regra central:** O Supervisor **nunca** executa código, acessa MCP ou gera SQL/PySpark.
Sempre delega. Agentes especialistas executam com seus MCPs pré-configurados.

---

## Estrutura de Diretórios (críticos)

```
agents/
  registry/       ← definições declarativas dos agentes (.md + YAML frontmatter)
  loader.py       ← carrega agentes do registry, resolve tool aliases
  supervisor.py   ← monta ClaudeAgentOptions com todos os agentes + hooks + MCPs
  prompts/        ← system prompt do Supervisor
  cache_prefix.md ← prefixo byte-idêntico injetado em TODOS os agentes (prompt caching)

mcp_servers/
  databricks/     ← MCP oficial Databricks (50+ tools)
  databricks_genie/ ← MCP customizado: Genie Conversation API
  fabric/         ← MCP oficial Microsoft Fabric
  fabric_community/ ← MCP comunidade: linhagem, dependências
  fabric_sql/     ← MCP customizado: SQL Analytics Endpoint via TDS
  fabric_rti/     ← MCP Fabric Real-Time Intelligence (KQL/Kusto)
  context7/       ← Docs atualizadas de bibliotecas (free, sem credenciais)
  tavily/         ← Busca web para LLMs (free: 1k créditos/mês)
  github/         ← GitHub: repos, issues, PRs (free via PAT)
  firecrawl/      ← Web scraping estruturado (free: 500 créditos/mês)
  postgres/       ← Queries readonly em PostgreSQL (free, open source)
  memory_mcp/     ← Knowledge graph de entidades (free, sem credenciais)
  migration_source/ ← MCP customizado: DDL/schema extraction de SQL Server/PostgreSQL
  _template/      ← Template para novos MCPs

config/
  settings.py     ← Pydantic BaseSettings — todas as credenciais + validação
  mcp_servers.py  ← Registry centralizado de MCP servers (ALL_MCP_CONFIGS)

hooks/            ← Hooks PreToolUse / PostToolUse
kb/               ← Knowledge Bases (referência, lida pelos agentes)
skills/           ← Skills operacionais (playbooks, lidos pelos agentes)
tests/            ← pytest — atualizar quando adicionar agentes/MCPs
```

---

## Como Adicionar um Novo Agente

**Crie `agents/registry/<nome>.md`** — o loader carrega automaticamente, sem tocar código Python.

```yaml
---
name: nome-do-agente
description: "Descrição objetiva. Use para: [casos de uso]. Invoque quando: [trigger]."
model: claude-sonnet-4-6        # T0/geral usa claude-haiku-4-5; T1/T2/T3 usam claude-sonnet-4-6
tools: [Read, Write, Grep, Glob, databricks_readonly, context7_all]
mcp_servers: [databricks, context7]
kb_domains: [databricks, sql-patterns]   # injeta index.md automaticamente
skill_domains: [databricks, patterns]    # injeta índice de SKILL.md disponíveis
tier: T2                                  # T0 | T1 | T2 | T3
---
# Nome do Agente

## Identidade e Papel
...
```

**Tiers:**
| Tier | Modelo padrão | maxTurns | Effort | Uso |
|------|---------------|----------|--------|-----|
| T0 | claude-haiku-4-5 | 3 | low | Conversacional puro, zero MCP — somente `geral` |
| T1 | claude-sonnet-4-6 | 20 | high | Core: pipelines complexos, multi-platform |
| T2 | claude-sonnet-4-6 | 12 | medium | Especializados: qualidade, governança, semântica |
| T3 | claude-sonnet-4-6 | 5 | low | Conversacionais com tools limitadas |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThomazRossito/data-agents](https://github.com/ThomazRossito/data-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

---
trigger: always_on
description: - É mandatório adicionar o prefixo `rtk` em todos os comandos.
---

# Guia para atuação de agentes de IA

## RTK

- É mandatório adicionar o prefixo `rtk` em todos os comandos.

## 1. Linguagem e contexto

- Sempre responder em **português**, a menos que o contexto da tarefa exija outro idioma (por exemplo, contrato externo em inglês).
- Este projeto é a camada **FastAPI** do __system_name__, organizada como API modular por domínios, usando **FastAPI**, **SQLAlchemy**, **Pydantic v2**, **PostgreSQL**, cache externo quando configurado, autenticação JWT compatível com o ecossistema Django existente e observabilidade via **Sentry**, **Elastic APM**, **Logfire** e logs da aplicação conforme configuração real do ambiente.
- O projeto foi derivado do ecossistema Django __system_name__ e ainda se integra a dados/modelos de autenticação e domínio compartilhados. Regras Django devem ser tratadas como contexto de integração, não como padrão técnico ativo desta camada.

## 2. Fontes de verdade para arquitetura

Ao gerar/alterar código, priorizar `.ia/docs/architecture/` (`overview.md`, `modules.md`, `ia_modules.md`, `security.md`) e `.ia/docs/guides/` (`patterns.md`, `constraints.md`), além de `.ia/docs/testing/strategy.md` para testes. Em caso de conflito, **`.ia/docs/` prevalece sobre este arquivo**.

## 3. Skills de IA por camada (obrigatório)

### 3.1. Localização das skills

Skills em `.ia/skills/<skill-name>/SKILL.md`.

### 3.2. Catálogo de skills disponíveis

Ordem do catálogo agrupa por finalidade (governança → FastAPI → IA/agent → especificação/integração → Obsidian).

| Skill | Quando ativar (trecho curto) |
| --- | --- |
| `workflow-demandas` | Pedido envolve tarefa, feature, bugfix, refactor, testes ou mudanças em `.ia/` |
| `branch-task-aprovada` | Task aprovada — abrir branch de implementação a partir do arquivo da task |
| `merge-com-dev` | Desenvolvedor pediu **merge com dev** ou merge da branch de task com dev |
| `task-encerramento` | Implementação concluída — mover task de `todo/` para `done/` |
| `governanca-compliance` | Auditoria de governança em `.ia/` ou validação antes de mudanças amplas |
| `atualizar-artefatos-ia` | Atualizar `AGENTS.md`, `.ia/` ou documentação operacional de agentes |
| `adicionar-endpoint` | Criar rota, endpoint ou CRUD FastAPI em módulo existente |
| `refatorar-modulo` | Refatorar módulo FastAPI para padrões arquiteturais vigentes |
| `corrigir-bug` | Diagnosticar erro, exceção, falha de teste ou comportamento inesperado |
| `escrever-testes` | Criar testes automatizados pytest/pytest-asyncio/TestContainers |
| `fastapi-tests-pytest` | Revisar ou escrever testes para routers, use cases e models FastAPI |
| `adicionar-embeddings` | Analisar ou especificar busca semântica, embeddings, pgvector, memória ou vetorização — não alterar módulo de IA/agentes do projeto (se existir) |
| `nova-tool-ia` | Especificar tool assíncrona para agentes — não alterar módulo de IA/agentes do projeto (se existir) |
| `novo-agente-ia` | Especificar agente especialista — não alterar módulo de IA/agentes do projeto (se existir) |
| `spec-integracao-flutter` | Criar spec de integração entre API FastAPI e cliente Flutter |
| `obsidian-sync` | Sincronizar repositório com vault DevBrain do Obsidian |
| `obsidian-query` | Responder pergunta sobre histórico/tasks/specs consultando vault DevBrain |

### 3.3. Regra de uso das skills

Antes de implementar: identificar camadas afetadas (router, schema, model SQLAlchemy, use case, dependência, integração, teste, governança), ler os `SKILL.md` correspondentes, aplicar as regras e registrar na task quais skills foram usadas. Mudança multicamada exige combinar `workflow-demandas` com as skills técnicas aplicáveis.

### 3.4. Precedência de aplicação

Mudança técnica em código ou documentação operacional: **primeiro** `workflow-demandas` para registrar o planejamento e o escopo; **depois** as skills por camada durante a execução. Pedidos operacionais isolados, como merge com `dev`, encerramento de task, sincronização com Obsidian ou auditoria de governança, podem acionar direto a skill correspondente.

## 4. Estrutura do projeto FastAPI

Mapa arquitetural amplo vive em `.ia/docs/architecture/overview.md` e `.ia/docs/architecture/modules.md`. A aplicação principal está em `main.py`, agrega routers em `core/routers.py` e expõe a API sob o prefixo configurado por `settings.api_str`.

**Padrão por módulo:** `routers.py` (rotas FastAPI) · `schemas.py` (contratos Pydantic) · `models.py` (modelos SQLAlchemy) · `use_cases.py` (orquestração/regras de aplicação) · arquivos auxiliares quando o módulo exigir integração, background task, uploads ou serviços externos.

**Regras transversais:**

- Routers devem permanecer finos; regra de aplicação deve viver em use cases, services ou helpers locais.
- Schemas Pydantic definem contrato de entrada/saída e devem evitar vazar detalhes internos desnecessários.
- Consultas SQLAlchemy devem evitar N+1, usar sessão sync/async adequada e manter filtros de `deleted`/`enabled` quando o modelo herdar de `CoreBase`.
- Dependências de autenticação/autorização devem reutilizar helpers de `authentication/security.py` quando aplicável.
- Integrações com Django, Flutter, cache, busca, Obsidian ou IA devem ser documentadas na task/spec quando alteradas.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgtecPalmas/AgtecCore](https://github.com/AgtecPalmas/AgtecCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

---
trigger: always_on
description: Antes de editar qualquer coisa neste projeto, siga a skill `supportfaq-project-navigator`.
---

# supportFAQagent — Claude Code Instructions

Antes de editar qualquer coisa neste projeto, siga a skill `supportfaq-project-navigator`.
Antes de commitar, fazer push ou abrir PR, siga a skill `supportfaq-git-flow`.
Quando alguém perguntar o que fazer agora, siga a skill `supportfaq-next-step-planner`.

---

## Skill: supportfaq-project-navigator

---
name: supportfaq-project-navigator
description: Use when an AI agent needs to understand, navigate, plan, or modify the supportFAQagent project without guessing architecture.
---

### O que esta skill faz

Skill que ajuda o agente a navegar no projeto sem alucinar arquitetura. Ela orienta a leitura do `README.md`, `CONTRIBUTING.md` e dos docs certos para cada tipo de mudança, indicando pastas, responsabilidades, testes e riscos de atropelar outra frente.

Use this skill before changing project structure, code, docs, domains, integrations, or knowledge content.

Goal: understand the project from repository sources, read only the needed docs, and avoid hallucinating architecture.

Also preserve the product positioning in `docs/product-positioning.md`: commercial but technical, operationally safe, traceable, and honest about MVP limits.

### First Step

Always start with:

1. Read `README.md`.
2. Read `docs/product-positioning.md` when the task touches README, docs, PR text, public positioning, onboarding, or agent instructions.
3. Read `CONTRIBUTING.md`.
4. Identify the change area.
5. Read only the docs needed for that area.

Do not load every doc by default.

### Mapa de documentação (onde achar cada coisa)

Primeiro destino: `docs/project-map.md` (estado das frentes) e `docs/navigation.md` (roteador por tarefa). Docs organizados por pasta:

| Pasta | O que vive aqui |
| --- | --- |
| `docs/` (raiz) | Índices: `project-map.md`, `navigation.md`, `documentation-status.md`, `product-positioning.md`, `agent-skills.md`, `references-legacy.md` |
| `docs/architecture/` | Design, fronteiras, contratos e padrões |
| `docs/setup/` | Guias de instalação/configuração de ambiente |
| `docs/MVP/` | Planos técnicos majoritários do MVP |
| `docs/quality-plans/` | Planos detalhados por frente |
| `docs/runbooks/` | Procedimentos operacionais |
| `docs/security/` | Planos e contratos de segurança |
| `docs/archive/` | Concluído, substituído ou obsoleto |

Lookup por assunto (ex.): Hermes → `docs/architecture/integration-contracts.md` (contrato) + `docs/quality-plans/hermes-chat-bridge-plan.md` (plano) + `docs/runbooks/hermes-chat-cutover.md` (operação). Caminhos antigos de docs movidos: `docs/references-legacy.md`.

### Area Map

| Change area | Read before editing | Likely folders |
| --- | --- | --- |
| Product positioning, README, public docs, PR narrative, agent instructions | `docs/product-positioning.md`, `README.md`, `docs/agent-skills.md` | `README.md`, `docs/`, `.agents/skills/` |
| Architecture or module boundaries | `docs/architecture/architecture.md`, `docs/MVP/technical-implementation-plan.md`, `docs/navigation.md` | `app/`, `docs/` |
| API contracts | `docs/architecture/integration-contracts.md`, `docs/architecture/observability.md` | `app/api/routes/`, `app/api/schemas/`, `app/feedback/`, `tests/` |
| Domain behavior | `docs/architecture/domain-contract.md`, `docs/navigation.md` | `domains/<domain>/domain.yaml`, `app/domain_engine/` |
| Knowledge base or FAQs | `docs/architecture/knowledge-authoring.md`, `docs/architecture/domain-evals.md` | `domains/<domain>/knowledge/`, `domains/<domain>/evals/` |
| Evals or calibration | `docs/architecture/domain-evals.md`, `docs/architecture/knowledge-authoring.md` | `app/evals/`, `domains/<domain>/evals/`, `tests/` |
| Ingestion | `docs/architecture/integration-contracts.md`, `docs/MVP/technical-implementation-plan.md` | `app/ingestion/`, `app/api/routes/ingestion.py`, `app/api/schemas/ingestion.py` |
| GitHub document loader | `docs/architecture/knowledge-authoring.md`, `docs/MVP/technical-implementation-plan.md`, `docs/navigation.md` | `app/ingestion/github_loader.py`, `scripts/fetch_github_document.py`, `tests/test_github_loader.py` |
| Retrieval or vector store | `docs/architecture/architecture.md`, `docs/MVP/technical-implementation-plan.md` | `app/retrieval/`, `app/orchestration/` |
| LLM/provider/prompt | `docs/MVP/technical-implementation-plan.md`, `docs/architecture/domain-contract.md` | `app/llm/`, `app/orchestration/`, `domains/<domain>/prompts/` |
| Handoff or escalation | `docs/architecture/domain-contract.md`, `docs/architecture/integration-contracts.md`, `docs/architecture/domain-evals.md` | `app/handoff/`, `app/orchestration/`, `domains/<domain>/domain.yaml`, `tests/` |
| Observability/logging | `docs/architecture/observability.md`, `docs/MVP/technical-implementation-plan.md` | `app/core/`, `app/main.py`, route files |
| Security or public surface hardening | `SECURITY.md`, `docs/security/`, `docs/architecture/observability.md`, `docs/architecture/code-standards.md` | `app/core/`, `app/api/`, `tests/security/`, `.github/workflows/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [renanfulas/supportFAQagent](https://github.com/renanfulas/supportFAQagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->

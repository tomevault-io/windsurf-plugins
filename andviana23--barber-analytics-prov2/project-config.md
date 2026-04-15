---
trigger: always_on
description: > O objetivo deste arquivo é fazer o Copilot/Codex atuar como um **desenvolvedor sênior do VALTARIS v1.0**, respeitando: arquitetura oficial, design system, modelo multi-tenant, banco Neon (PostgreSQL), fluxos de negócio críticos, integrações (principalmente Asaas) e evitando QUALQUER alteração que comprometa segurança, isolamento entre tenants ou consistência de dados.
---

> O objetivo deste arquivo é fazer o Copilot/Codex atuar como um **desenvolvedor sênior do VALTARIS v1.0**, respeitando: arquitetura oficial, design system, modelo multi-tenant, banco Neon (PostgreSQL), fluxos de negócio críticos, integrações (principalmente Asaas) e evitando QUALQUER alteração que comprometa segurança, isolamento entre tenants ou consistência de dados.

# Copilot/Codex — Manual Definitivo (Barber Analytics Pro v2.0)

Tom: firme, direto, técnico. Idioma: **pt-BR**. Se houver conflito com a documentação oficial, **a documentação vence**.

## 0. Leitura Obrigatória Antes de Codar

Para QUALQUER tarefa, antes de gerar, modificar ou remover código, o Copilot/Claude/Cursor deve OBRIGATORIAMENTE ler os documentos na seguinte ordem de prioridade:

### 1. Produto & Regras de Negócio
- `docs/07-produto-e-funcionalidades/PRD-VALTARIS.md`
- `docs/07-produto-e-funcionalidades/FLUXOS/*`
- `docs/07-produto-e-funcionalidades/CATALOGO_FUNCIONALIDADES.md`

### 2. Arquitetura do Sistema
- `docs/02-arquitetura/ARQUITETURA.md`
- `docs/02-arquitetura/DOMAIN_MODELS.md`
- `docs/02-arquitetura/MODELO_DE_DADOS.md`
- `docs/02-arquitetura/MODELO_MULTI_TENANT.md`
- `docs/02-arquitetura/FLUXOS_CRITICOS_SISTEMA.md`
- `docs/02-arquitetura/INTEGRACOES_EXTERNAS.md`

### 3. Backend
- `docs/04-backend/GUIA_DEV_BACKEND.md`
- `docs/04-backend/API_INTERNA.md`
- `docs/04-backend/API_PUBLICA.md`
- `docs/04-backend/EVENTOS_E_WEBHOOKS.md`
- `docs/04-backend/DTOs.md`
- `docs/04-backend/SERVICOS_E_MODULOS.md`
- `docs/04-backend/performance/*`

### 4. Frontend
- `docs/03-frontend/01-FOUNDATIONS.md`
- `docs/03-frontend/02-ARCHITECTURE.md`
- `docs/03-frontend/03-COMPONENTS.md`
- `docs/03-frontend/04-PATTERNS.md`
- `docs/03-frontend/MAPA_TELAS.md`
- `docs/03-frontend/DESIGN_SYSTEM.md`
- `docs/03-frontend/GUIA_FRONTEND.md`

### 5. Banco de Dados
- Sempre validar nomes de tabela e colunas via MCP `@pgsql`
- Nunca supor nomes
- Nunca criar SQL fora de repositórios
- Sempre respeitar tenant_id e RLS

### 6. Cálculos (FONTE DA VERDADE FINANCEIRA)
- Local: `docs/10-calculos/*`

Antes de implementar QUALQUER lógica relacionada a:
- comissões
- ocupação
- ticket médio
- metas
- ponto de equilíbrio
- churn
- LTV
- CAC
- precificação
- previsão de caixa
- DRE

O agente deve OBRIGATORIAMENTE:
1. Ler o arquivo correspondente dentro de `docs/10-calculos/`.
2. Usar EXATAMENTE a fórmula documentada.
3. Nunca “inventar” cálculos se o documento existir.
4. Se faltar fórmula → solicitar revisão do documento ANTES de codar.

### 7. Negócio e Métricas
- `docs/08-negocio-e-metricas/*`

### 8. Segurança
- `docs/06-seguranca/*`

### 9. Operações / SRE
- Logs
- Monitoramento
- Padrões de erro
- Estrutura de pastas
- etc.

**Regra de Ouro:**
Se houver conflito entre código existente e documentação, a prioridade é:
1. PRD
2. Fluxos
3. Cálculos
4. Arquitetura
5. Backend/Frontend
6. Código atual

---

## Ordem de leitura antes de atuar

Sempre consulte nesta ordem antes de sugerir mudanças relevantes (arquitetura, DB, regras de negócio, UI):

1. `docs/01-visao-geral/VISAO_GERAL_PRODUTO.md`
2. `docs/02-arquitetura/ARQUITETURA.md`
3. `docs/02-arquitetura/FLUXOS_CRITICOS_SISTEMA.md`
4. `docs/02-arquitetura/MODELO_DE_DADOS.md`
5. `docs/02-arquitetura/INTEGRACOES_EXTERNAS.md`
6. `docs/04-backend/GUIA_DEV_BACKEND.md`
7. `docs/03-frontend/DESIGN_SYSTEM.md`
8. `docs/03-frontend/GUIA_FRONTEND.md`
9. `docs/07-produto-e-funcionalidades/CATALOGO_FUNCIONALIDADES.md`
10. `docs/07-produto-e-funcionalidades/ROADMAP_PRODUTO.md`
11. `docs/06-seguranca/ARQUITETURA_SEGURANCA.md`
12. `docs/08-negocio-e-metricas/PLANOS_E_PRECOS.md`
13. `docs/02-arquitetura/ADR/*`
14. `docs/07-produto-e-funcionalidades/ASSINATURAS.md`
15. `docs/07-produto-e-funcionalidades/FINANCEIRO.md`
16. `docs/07-produto-e-funcionalidades/MANUAL_SUBSCRIPTION_FLOW.md`
17. `docs/07-produto-e-funcionalidades/ONBOARDING_FLOW_REVIEW.md`
18. `docs/07-produto-e-funcionalidades/ONBOARDING_WIZARD_IMPLEMENTATION.md`
19. `docs/07-produto-e-funcionalidades/PLANO_CONTINUACAO_ONBOARDING.md`
20. `docs/06-seguranca/COMPLIANCE_LGPD.md`
21. `docs/06-seguranca/RBAC.md`
22. `docs/11-Fluxos/FLUXO_*.md` (fluxos de negócio por módulo)

Se a tarefa depender de outro tópico, encontre a referência em `docs/` antes de propor qualquer alteração.

> Sempre que a tarefa envolver regra de negócio dos módulos core (agendamento, lista da vez, assinatura, financeiro, comissões, estoque, CRM, relatórios, permissões):  
> 1) Ler `PRD-VALTARIS.md`.  
> 2) Ler o fluxo correspondente em `docs/11-Fluxos/FLUXO_*.md`.  
> 3) Só então gerar ou alterar código.

---

## Stack oficial do projeto

- Backend: Go + sqlc + Clean Architecture.
- Frontend: Next.js 16 + TypeScript + App Router + Design System (MUI/Shadcn + tokens oficiais).
- Banco principal: **PostgreSQL (Neon)**.

Implicações para Copilot/Codex:

- Sempre gerar SQL para PostgreSQL.
- Nunca inventar sintaxe/extension sem checar; confirmar nomes via `@pgsql` antes de editar queries.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andviana23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

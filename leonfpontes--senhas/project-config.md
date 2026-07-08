---
trigger: always_on
description: Last Updated: 2026-06-27 (sessão 2)
---

# AGENTS.md - Guia Operacional para Agentes de IA

Last Updated: 2026-06-27 (sessão 2)
Project: Senhas / GiraHub - Multi-Tenant SaaS para emissão de tickets
Repository: leonfpontes/Senhas
Default Branch: master
Working Branch (atual): master
VPS: 76.13.231.19 (Hostinger) — projeto clonado em /opt/senhas

Este arquivo define como agentes de IA devem entender o sistema e como agir ao implementar mudanças com seguranca, qualidade e consistencia arquitetural.

---

## 1) Objetivo do Produto

Senhas e um SaaS multi-tenant para emissao e gestao de tickets (senhas) para atendimento em giras.

Principais modulos:
- API publica de emissao e reenvio de senha.
- Painel admin do tenant (giras, porta, tickets, analytics, config, auditoria).
- Painel platform (super admin) para gestao de tenants, usuarios globais, billing e feature flags.

---

## 2) Mapa Rapido do Monorepo

- backend/: FastAPI + SQLAlchemy async + Alembic + testes Pytest.
- frontend/: Next.js + TypeScript + Material UI + Jest/RTL.
- packages/shared-types: contratos tipados compartilhados.
- packages/shared-ui: componentes e tema compartilhado.
- docs/: arquitetura, API, auth, multi-tenancy, deploy e testes.
- e2e/: cenarios E2E.
- load_tests/: testes de carga.
- security/: scripts/checklist de seguranca.

---

## 3) Arquitetura e Regras Nao Negociaveis

### 3.1 Multi-tenancy (obrigatorio)

Toda operacao sensivel deve respeitar isolamento por tenant em 3 camadas:
1. JWT carrega tenant_id no payload.
2. Middleware coloca tenant_id em request.state.
3. Repository filtra por tenant_id em query.

Regra critica:
- Nenhuma leitura/escrita de entidade de tenant sem filtro explicito de tenant_id.
- Evite bypass de repository para logica de negocio, exceto quando realmente necessario e com filtro de tenant preservado.

### 3.2 Auth e autorizacao

- Roles principais: SUPER_ADMIN, ADMIN, OPERATOR.
- Endpoints admin so para escopo do tenant atual.
- Endpoints platform so para super admin (escopo global).

**Fluxo de autenticacao via cookie HttpOnly (desde 2026-06-27):**
- Login seta 3 cookies: `access_token` (HttpOnly, Secure, SameSite=Strict), `refresh_token` (HttpOnly), `auth_state=1` (nao-HttpOnly — legivel por JS para verificar login).
- `/auth/refresh` implementado: le `refresh_token` do cookie, valida com `decode_refresh_token` (requer `type=refresh`), emite novo access + rotaciona refresh.
- `jwt_middleware` extrai token do header `Authorization: Bearer` primeiro (impersonacao via sessionStorage), depois fallback para cookie `access_token`.
- `jwt_middleware` public_paths inclui `/auth/refresh`, `/auth/forgot-password`, `/auth/reset-password`.
- Frontend usa `withCredentials: true` no axios — nao ha token no header para sessoes normais.
- Impersonacao usa sessionStorage e header Bearer — fluxo preservado separado.
- `hasAuthToken()` checa: `sessionStorage.getItem('access_token')` OR `document.cookie.includes('auth_state=1')` OR `localStorage.getItem('user')`.
- Logout DEVE chamar `POST /api/v1/auth/logout` para limpar cookies no servidor.

### 3.3 Grupos de Permissao — OBRIGATORIO em toda funcionalidade

O sistema implementa RBAC fino via `PermissionGroup` / `GroupPermission`. Todo endpoint admin e toda
tela admin DEVEM respeitar esse sistema. Ignorar esse requisito e considerado um bug critico de seguranca.

#### Backend — todo novo endpoint admin precisa de:

```python
from src.models import PermissionFeature
from src.api.dependencies import require_group_permission

@router.get("/recurso", dependencies=[Depends(require_group_permission(PermissionFeature.FEATURE, "view"))])
@router.post("/recurso", dependencies=[Depends(require_group_permission(PermissionFeature.FEATURE, "insert"))])
@router.put("/recurso/{id}", dependencies=[Depends(require_group_permission(PermissionFeature.FEATURE, "edit"))])
@router.delete("/recurso/{id}", dependencies=[Depends(require_group_permission(PermissionFeature.FEATURE, "delete"))])
```

Acoes mapeadas por tipo de endpoint:
- GET (listagem/detalhe) → "view"
- POST (criar/registrar) → "insert"
- PUT/PATCH (atualizar) → "edit"
- DELETE (remover) → "delete"

Rotas existentes e suas features:
- Giras, Porta (door_control) → `PermissionFeature.GIRAS` / `PermissionFeature.PORTA`
- Tickets, tickets_bulk, validate_bulk, email_resend → `PermissionFeature.TICKETS`
- Mediuns → `PermissionFeature.MEDIUNS`
- Associados → `PermissionFeature.ASSOCIADOS`
- Usuarios → `PermissionFeature.USUARIOS`
- Estoque → `PermissionFeature.ESTOQUE`
- Mensalidades (financeiro/config/resumo/relatorio) → `PermissionFeature.FINANCEIRO`
- Contas a Pagar/Receber, Fluxo de Caixa, Config Financeira → `PermissionFeature.CONTAS_FINANCEIRAS`
- Configuracoes do Tenant → `PermissionFeature.CONFIGURACOES`
- Auditoria → `PermissionFeature.AUDITORIA`
- Analytics → `PermissionFeature.ANALYTICS`
- Relatorio de Gira / exports CSV → `PermissionFeature.RELATORIO_GIRA`
- Cursos Presenciais / Sites → `PermissionFeature.CURSOS_PRESENCIAIS`

Para nova feature sem equivalente existente:
1. Adicionar valor ao enum `PermissionFeature` em `backend/src/models/permission_groups.py`.
2. Criar migracao Alembic para adicionar o valor ao tipo ENUM no banco (`ALTER TYPE ... ADD VALUE`).
3. Adicionar entrada em `frontend/src/constants/permissionFeatures.ts` (type union + FEATURE_LABELS com label e group).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonfpontes/Senhas](https://github.com/leonfpontes/Senhas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->

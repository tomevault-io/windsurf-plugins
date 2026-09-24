---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # servidor de desenvolvimento (Next.js)
npm run build    # build de produção
npm run start    # inicia servidor de produção
npm run hash <senha>  # gera hash bcrypt para cadastrar usuário manualmente
```

Não há test runner configurado. Não há linter configurado (sem eslint/prettier nos scripts).

## Variáveis de Ambiente

Copie `.env.example` para `.env` e preencha:

- `DATABASE_URL` — PostgreSQL principal (propostas, usuários, tabelas legadas `TCE_*`)
- `SCHEDULING_DATABASE_URL` — PostgreSQL de agendamentos (opcional; fallback para `DATABASE_URL`)
- `JWT_SECRET` — string aleatória ≥ 64 chars
- `CRON_SECRET` — segredo para o endpoint `GET /api/cron/cleanup-sessions`
- Variáveis `SMTP_*` — Microsoft 365, STARTTLS na porta 587
- Variáveis `ERP_*`, `NOTIFICATION_RECIPIENTS`, `NEXT_PUBLIC_ENABLE_VINCULO_VI` — configuração por filial (ver abaixo)

## Multi-filial (Viana / Varginha)

O mesmo codebase atende múltiplas filiais, **cada uma como um deploy próprio**
(projeto Vercel próprio) apontando para **seu próprio banco Supabase**. O banco
do portal é single-tenant (sem coluna de filial): senha sequencial, limites do
dia, usuários e regras são isolados por instância naturalmente.

As diferenças entre filiais ficam em `src/shared/infra/config/erpConfig.ts`,
vindas de env (defaults = Viana):

| Env | Viana (default) | Varginha |
|---|---|---|
| `ERP_FILIAL` | `VI` | `VA` |
| `ERP_ID_CLIENTE` / `ERP_ID_OPERACAO` / `ERP_ID_REFERENCIA` | `18` / `2` / `9999` | conforme Protheus |
| `ERP_REQUIRE_PROPOSAL_FORNECEDOR` | `false` | `true` |
| `NOTIFICATION_RECIPIENTS` | equipe Viana | equipe Varginha |
| `NEXT_PUBLIC_ENABLE_VINCULO_VI` | `true` | `false` |

**Origem dos dados por filial** (o ETL entrega as tabelas-espelho no MESMO
formato; a tradução acontece no ETL, não no portal):

- **Viana**: propostas fechadas → `tce_operacoesdiarias`; fornecedores/corretores
  no `clifor`/`corretores`; fornecedor do agendamento resolvido pelo vínculo
  VI × Fornecedor (módulo `vi-fornecedor`).
- **Varginha**: compras do Protheus (SC7) → `tce_operacoesdiarias` **com a coluna
  `id_fornecedor` já resolvida** (C7_FORNECE + C7_LOJA → `clifor.idclifor`);
  SA2 → `clifor` (fornecedores E corretores; `corretores` é uma VIEW sobre
  `clifor.is_corretor`). O módulo `vi-fornecedor` não é usado (menu oculto).
  Schema dos espelhos: `scripts/va-protheus-mirror.sql`. Em Viana, rodar
  `scripts/vi-add-id-fornecedor.sql` (coluna nova, permanece NULL).

**Resolução do fornecedor no agendamento** (POST `/api/scheduling` e
`/api/agendamento-manual`): `body.idFornecedor` > vínculo VI > `proposal.idFornecedor`
(espelho do ERP) > lookup por nome na `clifor`. Com
`ERP_REQUIRE_PROPOSAL_FORNECEDOR=true`, o último fallback é desativado e a
ausência de fornecedor vira erro claro (evita resolver o corretor como
fornecedor silenciosamente).

## Arquitetura

### Clean Architecture + DDD por módulo

Cada módulo em `src/modules/` segue a estrutura:

```
modules/<domínio>/
  domain/
    entities/       # entidades e value objects puros
    errors/         # erros de domínio (estendem DomainError)
    repositories/   # interfaces (IXxxRepository)
    rules/          # regras de negócio puras
  application/
    use-cases/      # classes com responsabilidade única
  infra/
    repositories/   # implementações PostgreSQL (PgXxxRepository)
    mappers/        # conversão DB row → entidade
    index.ts        # instancia e exporta dependências (DI manual)
```

**Módulos existentes:** `user`, `scheduling`, `delivery-rules`, `vi-fornecedor`

### Injeção de dependência

Não há container IoC. Cada módulo tem um `infra/index.ts` (ex.: `userDependencies`, `schedulingDependencies`) que instancia repositórios e use cases manualmente. As rotas de API importam dessas factories.

### Rotas de API

Todas as rotas ficam em `src/app/api/`. O padrão de autenticação em toda rota protegida:

```ts
const user = requireAuth(request)           // ou requireRole / requireAnyRole
if (!isAuthenticated(user)) return user     // user é NextResponse 401/403

// user.userId, user.email, user.roles disponíveis
```

Guards em `src/shared/infra/auth/authGuard.ts`. Token JWT extraído do header `Authorization: Bearer <token>`.

### Autenticação cliente

`src/shared/presentation/auth/` contém:
- `authFetch` — wrapper de `fetch` que injeta o access token e refaz a chamada automaticamente após `/api/auth/refresh` em respostas 401
- `tokenStorage` — mantém o access token em memória (não em localStorage/cookie)
- `userStorage` — persiste info do usuário em `localStorage`

O refresh token fica em cookie HttpOnly; o access token (JWT 15 min) fica só em memória no cliente.

### Banco de dados

Dois clientes `postgres` instanciados em `src/shared/infra/database/`:
- `db` — aponta para `DATABASE_URL`
- `schedulingDb` — aponta para `SCHEDULING_DATABASE_URL` (ou `DATABASE_URL` se ausente)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [realCafeMiguel/agendamento](https://github.com/realCafeMiguel/agendamento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

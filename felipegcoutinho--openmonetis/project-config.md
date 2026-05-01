---
trigger: always_on
description: > Self-hosted personal finance app (Next.js 16, React 19, PostgreSQL, Drizzle ORM, Better Auth, Tailwind 4, shadcn/ui).
---

# CLAUDE.md - OpenMonetis

> Self-hosted personal finance app (Next.js 16, React 19, PostgreSQL, Drizzle ORM, Better Auth, Tailwind 4, shadcn/ui).
> Portuguese UI, English folders/imports. Linter: Biome 2.x. Package manager: pnpm.

## Related Projects

- **OpenMonetis Companion** (`~/github/openmonetis-companion`): Android app que captura notificacoes de apps bancarios e envia para o OpenMonetis via API. Os itens chegam na feature `inbox` para revisao.

---

## Critical Rules

1. **Sempre filtrar por `userId`** em queries.
2. **Usar `getAdminPayerId(userId)`** de `src/shared/lib/payers/get-admin-id.ts` ao inves de JOIN com `payers` para descobrir o admin.
3. **Periods** usam formato `YYYY-MM` (ex: `"2025-11"`). Utils em `src/shared/utils/period/`.
4. **Moeda**: R$ com 2 decimais. DB: `numeric(12, 2)`. Utils em `src/shared/utils/currency.ts`.
5. **Revalidation**: usar `revalidateForEntity("entity")` de `src/shared/lib/actions/helpers.ts` apos mutations.
6. **Versionamento**: registrar mudancas no `CHANGELOG.md` seguindo Keep a Changelog, também altere o `package.json` e `readme.md` (Badges do README.md). Cada versão deve ter um parágrafo introdutório em linguagem humana logo abaixo do cabeçalho `## [x.y.z]`, antes das seções `### Adicionado/Alterado/Removido` — descrevendo em prosa o que a versão representa (ex: "Esta versão foca em polimento visual e reorganização interna...").
7. **Comunicacao**: responder em portugues clara e direta com o time.
8. **Commit messages**: agrupar por natureza. em pt-br. seguindo o padrao do sistema.
9. **README.md**: sempre que fizer alteracoes significativas, atualize o README.md.

---

## Architecture

### Feature-First

- `src/app/`: roteamento, layouts, loading states e paginas finas
- `src/features/`: codigo de dominio por feature
- `src/shared/`: tudo que e genuinamente reutilizado entre features
- `src/db/`: schema do banco

### Regra Feature vs Shared

Use esta pergunta:

> Se eu deletar esta feature, este arquivo deveria sumir junto?

- Sim: vai para `src/features/<feature>/`
- Nao: vai para `src/shared/`

### Features nao importam outras features

Se um contrato cruza dominios, ele deve morar em `src/shared/`.

**Excecao intencional: `attachments` depende de `transactions`**

`src/features/attachments` importa `TransactionDialog`, `TransactionDetailsDialog` e `TransactionItem` diretamente de `src/features/transactions`. Isso e uma dependencia explicita e aceita: anexos sao semanticamente uma extensao de lancamentos — existem por causa deles e nao fazem sentido sem esse contexto. Mover esses componentes para `shared/` seria errado (eles pertencem a transactions). Nao tratar isso como bug a corrigir.

Exemplos comuns:

- auth: `src/shared/lib/auth/*`
- db: `src/shared/lib/db.ts`
- revalidation helpers: `src/shared/lib/actions/*`
- payers cross-domain helpers: `src/shared/lib/payers/*`
- period/currency/date: `src/shared/utils/*`
- shadcn/ui: `src/shared/components/ui/*`

---

## Directory Structure

```text
src/
├── app/
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   └── signup/page.tsx
│   ├── (dashboard)/
│   │   ├── dashboard/
│   │   ├── transactions/
│   │   ├── cards/
│   │   │   └── [cardId]/invoice/
│   │   ├── accounts/
│   │   │   └── [accountId]/statement/
│   │   ├── categories/
│   │   │   ├── [categoryId]/
│   │   │   └── history/
│   │   ├── budgets/
│   │   ├── payers/
│   │   │   └── [payerId]/
│   │   ├── notes/
│   │   ├── insights/
│   │   ├── calendar/
│   │   ├── inbox/
│   │   ├── attachments/
│   │   ├── changelog/
│   │   ├── reports/
│   │   │   ├── category-trends/
│   │   │   ├── card-usage/
│   │   │   ├── installment-analysis/
│   │   │   └── establishments/
│   │   └── settings/
│   ├── (landing-page)/
│   ├── api/
│   ├── globals.css
│   └── layout.tsx
├── features/
│   ├── auth/
│   ├── landing/
│   ├── dashboard/
│   ├── transactions/
│   ├── cards/
│   ├── invoices/
│   ├── accounts/
│   ├── categories/
│   ├── budgets/
│   ├── payers/
│   ├── notes/
│   ├── insights/
│   ├── calendar/
│   ├── inbox/
│   ├── attachments/
│   ├── reports/
│   └── settings/
├── shared/
│   ├── components/
│   │   ├── ui/
│   │   ├── navigation/
│   │   ├── providers/
│   │   ├── month-picker/
│   │   ├── logo-picker/
│   │   ├── calculator/
│   │   ├── entity-avatar/
│   │   └── skeletons/
│   ├── hooks/
│   ├── lib/
│   │   ├── actions/
│   │   ├── auth/
│   │   ├── accounts/
│   │   ├── cards/
│   │   ├── calculator/
│   │   ├── categories/
│   │   ├── email/
│   │   ├── installments/
│   │   ├── invoices/
│   │   ├── logo/
│   │   ├── payers/
│   │   ├── schemas/
│   │   ├── transfers/
│   │   ├── types/
│   │   └── db.ts
│   └── utils/
│       ├── period/
│       ├── currency.ts
│       ├── date.ts
│       ├── financial-dates.ts
│       ├── percentage.ts
│       ├── category-colors.ts
│       ├── calendar.ts
│       ├── math.ts
│       ├── number.ts
│       ├── string.ts
│       ├── initials.ts
│       ├── icons.tsx
│       ├── export-branding.ts
│       ├── ui.ts
│       └── calculator.ts
└── db/
    └── schema.ts
```

---

## Import Patterns

### Preferidos

```ts
import { getUser } from "@/shared/lib/auth/server";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felipegcoutinho/openmonetis](https://github.com/felipegcoutinho/openmonetis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

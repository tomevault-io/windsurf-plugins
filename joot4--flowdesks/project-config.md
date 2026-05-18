---
trigger: always_on
description: > Esse arquivo é a fonte de verdade para o agente de IA entender o projeto, manter o padrão e evitar regredir decisões já tomadas. **Sempre leia antes de editar código.**
---

# FlowDesks — Contexto do Projeto

> Esse arquivo é a fonte de verdade para o agente de IA entender o projeto, manter o padrão e evitar regredir decisões já tomadas. **Sempre leia antes de editar código.**

---

## 1. Visão geral

**FlowDesks** é um PWA fullstack para gestão operacional de equipes em campo: calendário, alocações, controle de ponto com geofence, evidências fotográficas, ajustes de ponto e exportação para pagamento.

- **Frontend:** Angular 17 standalone + Signals + Reactive Forms + Angular Material + FullCalendar + Tailwind
- **PWA:** `@angular/service-worker` + `ngsw-config.json`
- **Backend:** Supabase (Auth + Postgres + RLS + RPC + Edge Functions + Storage)
- **Timezone:** `America/Fortaleza` no frontend, `timestamptz` em UTC no banco
- **Idiomas:** pt-BR, en, es (default atual: en) via `TranslatePipe` (`| t`)
- **Identificador da app no Supabase:** `profiles.app_code = 'FLOWDESKS'` — usuários de outras apps são bloqueados no login
- **Isolamento físico (migration 030):** TODAS as tabelas operacionais (`assignments`, `employees`, `locations`, `activity_types`, `assignment_attendances`, `assignment_attendance_requests`, `assignment_work_photos`, `notifications`, `reassignment_logs`) têm `app_code text not null default 'FLOWDESKS' check (app_code = 'FLOWDESKS')` + trigger BEFORE INSERT que força o valor. Helper `public.is_flowdesks_user()` é usado em toda RLS. Rollback em `supabase/sql/030_flowdesks_lockdown_rollback.sql`

---

## 2. Perfis e roteamento

Definidos em `src/app/shared/models/role.model.ts`:

| Role          | Home          | Shell                                              |
| ------------- | ------------- | -------------------------------------------------- |
| `SUPER_ADMIN` | `/director`   | `features/director/director.shell.ts`              |
| `ADMIN`       | `/admin`      | `features/admin/admin.shell.ts`                    |
| `COLLABORATOR`| `/me/ponto`   | `features/collaborator/collaborator.shell.ts`      |

Guards: `core/guards/auth.guard.ts` e `core/guards/role.guard.ts`. Rotas em `src/app/app.routes.ts`.

---

## 3. Estrutura de diretórios

```
src/app/
  app.component.{ts,html,scss}     # toolbar global, banner offline, idioma, logout
  app.config.ts                    # providers (router, animations, sw, snackbar)
  app.routes.ts                    # rotas + guards por role
  core/
    guards/                        # authGuard, roleGuard
    i18n/i18n.service.ts           # dicionário pt-BR/en/es + signal de idioma
    supabase/                      # client, session.store (signals), services e RPCs
    ui/                            # toast.service, pwa-notification.service
  shared/
    components/                    # confirm-dialog, loading
    models/                        # assignment.model.ts, role.model.ts
    pipes/                         # t.pipe (i18n), tz-date.pipe (Fortaleza)
  features/
    auth/login.page.ts
    admin/
      admin.shell.ts               # nav superior do ADMIN
      calendar/                    # FullCalendar + diálogos (assignment, reassign, payment-summary)
      employees/                   # CRUD colaboradores
      catalogs/                    # locations.page, activity-types.page
      work-photos/                 # galeria de fotos com filtros + ZIP
    collaborator/
      collaborator.shell.ts        # apenas <router-outlet/>
      my-calendar.page.ts          # "ponto" (check-in/out + geofence)
      requests.page.ts             # solicitações de ajuste
      profile.page.ts
      assignment-detail.dialog.ts
    director/
      director.shell.ts
      admins.page.ts

supabase/
  migrations/                      # SQL versionado (já em 029_*)
  functions/create-collaborator/   # Edge Function (Service Role) p/ criar usuário no Auth
  sql/                             # scripts auxiliares (make_admin, seed, ...)
```

---

## 4. Padrões obrigatórios

### Angular
- **Sempre standalone**, com `imports: [...]` no decorator. Nunca criar NgModule.
- **Sempre `ChangeDetectionStrategy.OnPush`** em componentes novos.
- **Estado em Signals** (`signal`, `computed`, `effect`). Evitar `BehaviorSubject` se não houver razão forte.
- **Reactive Forms tipados** via `formBuilder.nonNullable.group(...)`.
- **Templates** preferem control flow novo (`@if`, `@for`, `@switch`). Não usar `*ngIf`/`*ngFor` em código novo.
- **Lazy load por rota** com `loadComponent: () => import(...).then(m => m.X)`.
- **i18n:** todo texto visível usa `{{ 'chave' | t }}`. Adicionar a mesma chave nos 3 idiomas em `i18n.service.ts`.
- **Datas no UI:** sempre via `tzDate` pipe ou formatadas com `Intl.DateTimeFormat` em `America/Fortaleza`.
- **Naming:**
  - serviços `*.service.ts`, dialogs `*.dialog.ts`, páginas `*.page.ts`, shells `*.shell.ts`, models `*.model.ts`
  - seletores com prefixo `app-` (configurado em `angular.json`)

### Estilos
- **Material 17** com tema custom (cyan + teal) em `src/styles.scss`.
- **Tailwind** habilitado (`tailwind.config.js`) com paleta `brand.50..900` e shadow `soft`. Usar Tailwind para utilitários; Material para componentes.
- **Tokens CSS globais** em `:root`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Joot4/flowdesks](https://github.com/Joot4/flowdesks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

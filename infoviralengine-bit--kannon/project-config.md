---
trigger: always_on
description: > Documento di contesto vivo per Claude Code (e per chiunque legga la repo).
---

# CLAUDE.md · Kannon Hub Context

> Documento di contesto vivo per Claude Code (e per chiunque legga la repo).
> Aggiornalo ogni volta che cambia qualcosa di strutturale: nuova entità DB, nuova RPC, nuovo ruolo, nuova route, nuovo modulo applicato.
> **Convenzione**: questo file è la fonte di verità per le decisioni architetturali. Se contraddice un singolo file di codice, vince questo.

---

## 1. Cos'è Kannon

Kannon è un'agenzia B2B di user acquisition su TikTok per app mobile. Schiera decine/centinaia di creator su account TikTok freschi e niche-locked per generare views verso campagne dei clienti (Finanz, Easy Papiro, HAT Music, Tot Money, ecc.). I clienti pagano un fisso mensile + un CPM variabile. Kannon paga i creator un fisso + CPM proporzionato.

Il **Kannon Hub** (questa repo) è la piattaforma operativa interna che gestisce:
- pianificazione contenuti (calendario brief),
- esecuzione (creator portal),
- approvazione (client portal),
- scraping performance (Apify TikTok),
- pagamenti (entrate clienti + uscite creator),
- finance unificata,
- analytics multi-dimensionale.

URL produzione: `hub.thekannon.io`. URL landing pubblica: `thekannon.io`.

---

## 2. Stack tecnico

- **Frontend**: React 18 + TypeScript + Vite. UI da shadcn/ui + Tailwind CSS. State server: TanStack Query (react-query v5). Routing: react-router-dom v6. Charts: Recharts.
- **Backend**: Supabase (Postgres + Auth + RLS + Edge Functions in Deno + Storage). Tutte le query passano dal client `@/integrations/supabase/client`.
- **Scraping**: Apify (`clockworks/free-tiktok-scraper`). Edge function `scrape-tiktok` orchestra runs + polling background + ingestion (SP#5: no più webhook come meccanismo primario).
- **Deploy**: Lovable storicamente (preview). Ora il workflow è Claude Code → git push → Lovable preview / Vercel production.

Branding colori (per UI/landing, NON nel hub):
- bg `#DFDFDF`, ink `#000000`, accent `#FF2727`, accent-2 `#FFD0D0`.

Font (landing, NON hub): Inter (body), Instrument Serif italic (display accents), JetBrains Mono (eyebrow labels uppercase).

---

## 3. RBAC: i 7 ruoli

Definiti in `src/lib/roles.ts` come constants. Mai magic string. Usare sempre `ROLES.X` e `ROLE_GROUPS.Y`.

| Ruolo | Cosa fa | Default landing | Sidebar |
|---|---|---|---|
| `admin` | Owner della piattaforma. Accesso totale. | `/dashboard` | Tutte le sezioni + Finance + Settings |
| `team` | Operatori interni. Come admin ma senza Finance/Settings. | `/dashboard` | Tutto eccetto Finance/Settings |
| `outreach` | Single-purpose: solo Recruiting. | `/dashboard/recruiting` | Solo Recruiting |
| `closer` | Single-purpose: solo Closer. | `/dashboard/closer` | Solo Closer |
| `campaign_manager` | Pianifica contenuti, gestisce calendario, analytics format. | `/dashboard/content-calendar` | Campagne · Video Analytics · Calendario Contenuti |
| `creator` | Talento esterno. Solo portale dedicato. | `/creator` | Portale custom (CreatorArea) |
| `client` | Cliente Kannon. Solo portale dedicato. | `/client` | Portale custom (ClientArea) |

Helper functions:
- `canAccess(role, allowedRoles[])` — boolean check
- `isStaff(role)` — true se admin/team
- `ROLE_DEFAULT_ROUTE[role]` — landing post-login
- `ROLE_GROUPS.STAFF` = admin + team
- `ROLE_GROUPS.INTERNAL` = admin + team + outreach + closer + campaign_manager
- `ROLE_GROUPS.SINGLE_PURPOSE` = outreach + closer (NB: campaign_manager rimosso dopo SP#4)
- `ROLE_GROUPS.EXTERNAL` = creator + client

Le RLS Postgres sono **enforcement primario**. Le restrizioni UI sono solo UX. Mai fidarsi di gate solo lato client.

---

## 4. Entità DB principali

Mappa concettuale. Per schema completo consultare `supabase/migrations/`. Le migrations sono ordinate per timestamp.

### Auth e profili
- `auth.users` (Supabase native)
- `profiles` → 1:1 con `auth.users`, contiene full_name, role, ecc.
- `user_roles` → tabella ruoli (per `has_role()` SQL function)

### Business core
- `campaigns` → name, client_name, client_user_id (FK a auth.users), client_fixed, client_cpm, video_views_cap, monthly_spend_cap, start_date, status, **brief_threshold_views**, **brief_threshold_engagement**
- `creators` → name, status, user_id (FK), creator_fixed, creator_cpm, min_videos_per_day
- `campaign_creators` → junction N:N campagne ↔ creator
- `tiktok_accounts` → username, account_type ('creator'|'brand'|...), campaign_id, creator_id, is_active, last_scraped_at
- `contracts` + `contract_campaigns` + `contract_creators` + `contract_signatures` → contratti creator-Kannon, periodi di attività

### Video e performance
- `videos` → tiktok_account_id, tiktok_video_id (UNIQUE), views, likes, comments, published_at, window_expires_at, window_closed, views_final, last_scraped_at, **audio_id**, **audio_name**, **caption**, **hashtags[]**, content_tag
- `scraping_logs` → run_at, status (CHECK `running`/`success`/`error`), accounts_processed, videos_created, videos_updated, error_message, **run_id, dataset_id, started_at, completed_at, progress_note, triggered_by** (SP#5: status sync per polling background)

### Pagamenti

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [infoviralengine-bit/kannon](https://github.com/infoviralengine-bit/kannon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

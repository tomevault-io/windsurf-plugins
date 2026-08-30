---
trigger: always_on
description: Ferramenta SaaS para criação e publicação automatizada de anúncios no Meta Ads (Facebook/Instagram). Integra com a Meta Graph API **v25.0** via Supabase Edge Functions. Usuários são gestores de tráfego que sobem campanhas rapidamente sem abrir o Gerenciador de Anúncios. Quatro presets de campanha:
---

# F3F AUTO-ADS — Guia para Claude

## O que é este projeto

Ferramenta SaaS para criação e publicação automatizada de anúncios no Meta Ads (Facebook/Instagram). Integra com a Meta Graph API **v25.0** via Supabase Edge Functions. Usuários são gestores de tráfego que sobem campanhas rapidamente sem abrir o Gerenciador de Anúncios. Quatro presets de campanha:

- **FASE 1** — Tráfego para perfil do Instagram (`OUTCOME_TRAFFIC`, `PROFILE_VISIT`, `INSTAGRAM_PROFILE`)
- **FASE 2** — Engajamento de vídeo p/ montar públicos (`THRUPLAY`, `ON_VIDEO`) — alimenta audiências VV
- **FASE 3** — Leads via WhatsApp (`OUTCOME_LEADS`, `CONVERSATIONS`, `WHATSAPP`); variante **Vendas** usa `OUTCOME_SALES`
- **L.T** — Tráfego p/ site / conversões (`OFFSITE_CONVERSIONS`, `WEBSITE`) com nomenclatura própria

## Stack

| Camada | Tecnologia |
|--------|-----------|
| Frontend | React 18.3 + TypeScript 5.8 + Vite 5.4 (plugin SWC) |
| Roteamento | React Router 6.30 (páginas lazy) |
| Estado servidor | TanStack Query 5.83 |
| Formulários | React Hook Form 7.61 + Zod 3.25 |
| UI | shadcn/ui (Radix) + Tailwind 3.4 (`tailwindcss-animate`) |
| Auth | Supabase Auth (email/senha) — `@supabase/supabase-js` 2.98 |
| Backend | Supabase Edge Functions (Deno/TypeScript) |
| Testes | Vitest 3.2 + Testing Library + jsdom |
| API Meta | Meta Graph API **v25.0** (nunca usar outra versão) |

## Estrutura de diretórios

```
src/
  pages/          # Index, LoginPage, AdminPage, ResetPasswordPage, SettingsPage, MetaCallback, NotFound
  components/     # PublishForm (core, ~2450 linhas; contém child LogPanel)
                  # Header, Fase3Components, LocationSelector, SearchableSelect,
                  # IDDisplay, NavLink, ProtectedRoute, + ui/ (48 componentes shadcn)
  contexts/       # AuthContext (Supabase), ThemeContext (dark/light)
  hooks/          # use-toast, use-mobile
  lib/            # meta-api.ts (client das edge functions), naming.ts (geradores de nome),
                  # admin.ts (isCurrentUserAdmin / inviteUser), utils.ts (cn)
  integrations/   # supabase/client.ts + supabase/types.ts (gerados)

supabase/functions/   # 21 functions + _shared/
  # ── Publicação ──
  meta-publish/             # Edge principal — orquestra campaign > adset > creative > ad (1 chamada por criativo)
  meta-publish-validate/    # Pré-voo: cria adset de teste, valida targeting/budget, limpa
  meta-validate-creative/   # Valida link IG (media_id) ou arquivo do Drive (Range via API key)
  # ── OAuth / Token ──
  meta-login/               # Redirect OAuth Meta (scopes incl. whatsapp_business_management)
  meta-oauth-callback/      # Troca code → token curto → token longo; grava em meta_connections
  meta-status/              # Checa validade/expiração do token; auto-refresh se faltam ≤7 dias
  meta-token-refresh-cron/  # Cron diário: renova tokens que expiram em ≤14 dias (CRON_SECRET)
  # ── Descoberta (popula dropdowns) ──
  meta-ad-accounts/         # Lista contas + descobre IG accounts via páginas
  meta-audiences/           # Públicos custom + saved (paginado)
  meta-campaigns/           # Campanhas ACTIVE (detecta CBO vs ABO)
  meta-whatsapp-numbers/    # Descobre números WABA (promote_pages como fonte primária)
  meta-message-templates/   # Templates WhatsApp extraídos de creatives CTWA existentes
  meta-pixels/              # Lista pixels da conta
  meta-location-search/     # Busca geográfica
  # ── FASE 2 ──
  meta-create-video-audience/  # Cria custom audience VV50% de um vídeo
  # ── Diagnóstico (debug, não-produção) ──
  meta-ad-review/           # Lê ad_review_feedback + effective_status p/ diagnosticar rejeição
  meta-adset-diff/          # Compara configs de adsets
  meta-campaign-diagnostic/ # Dump de estrutura campaign/adset/ad/creative
  meta-fase1-diagnostic/
  meta-fase3-diagnostic/    # Testa variações de promoted_object WhatsApp (PAUSED, limpa)
  # ── Admin ──
  admin-invite-user/        # Cria gestor no Auth + dispara email de convite (requer app_admins)
  _shared/email.ts          # Helper Resend (sendInviteEmail)
```

## Fluxo de publicação (`meta-publish`)

```
1. Validar identidade (page_id + instagram_actor_id)
2. Resolver mídia por criativo → buildOne() mapeado em paralelo (1 chamada cada)
3. Criar Campaign (ACTIVE) — checkpoint manual removido (2026-07-06); entrega imediata ao publicar
4. Criar Adset (ACTIVE) — com promoted_object correto por preset
   (FASE 2 Completo: 1 criativo + N adsets, um por audiência; FASE 2 Adaptado: N criativos + 1 adset por criativo, cada um combinando 2-10 públicos)
5. Criar Adcreative
6. Criar Ad (ACTIVE)
```

### Resolução de mídia

- **IG shortcode/reel** → `source_instagram_media_id` (query direta na media do IG account)
- **Drive vídeo** → **primário**: `GOOGLE_DRIVE_API_KEY` + `file_url` no FormData → **Meta baixa o arquivo direto** (evita OOM 546 na edge). Faz poll do status do vídeo **5x** no fast-path `file_url` (index.ts:687) / **4x** no fallback multipart (index.ts:807).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agenciaf3f-droid/f3f-auto-ads](https://github.com/agenciaf3f-droid/f3f-auto-ads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->

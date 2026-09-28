---
trigger: always_on
description: Sistema all-in-one de reputação, fidelidade e relacionamento direto com clientes para pequenos negócios locais na Irlanda. Combina hardware físico impresso em 3D com plataforma SaaS multi-tenant.
---

# SmartTap — Contexto Completo do Projeto

## O QUE É O SMARTTAP

Sistema all-in-one de reputação, fidelidade e relacionamento direto com clientes para pequenos negócios locais na Irlanda. Combina hardware físico impresso em 3D com plataforma SaaS multi-tenant.

**Tagline:** "Você já pagou para trazer esse cliente uma vez. O SmartTap faz ele voltar."
**Tagline produto:** "TAP. CONNECT. GROW."
**Domínio:** smarttap.ie
**Fundador:** Henrique — indie developer baseado em Dublin, Irlanda

---

## OS 3 PILARES

- **Tap & Review:** NFC/QR → Google Review em 1 clique, sem app
- **Tap & Return:** Stamps digitais sem app → recompensas por fidelidade
- **Tap & Connect:** Coleta contato GDPR-compliant → campanhas diretas

---

## HARDWARE FÍSICO

- Impressora 3D Creality Hi CFS multi-color em Dublin
- 400 NFC tags disponíveis imediatamente
- Custo de produção: ~€0,55 por unidade | Preço: €79-€249 setup
- **3 formatos:**
  - Counter Stand (balcão) — 80x80x60mm
  - Table Tent (mesa) — 90x60mm triangular
  - Wall Plaque (parede) — 120x80mm
- **10 cores PLA:** Preto, Branco, Cinza, Navy, Roxo, Vermelho, Amarelo, Stone Age, Real Green, Forest Green

---

## IDENTIDADE VISUAL (NÃO ALTERE SEM APROVAÇÃO)

> ⚠️ **Rebrand em curso (aprovado 2026-05-31): "SmartTap Dark Electric".** A identidade antiga (verde/âmbar/cream + DM Serif) foi substituída pela paleta dark + ciano elétrico, feel Linear/Vercel. Rollout sequencial: **landing → dashboard → NFC → emails → stand físico**. A landing é a primeira superfície (ver `docs/superpowers/specs/2026-05-31-landing-dark-electric-redesign-design.md`). Superfícies ainda não migradas continuam na paleta antiga até serem feitas.

**Identidade ATUAL (Dark Electric):**
- **Logo:** símbolo ST com ondas NFC — **ciano #00D4FF / branco** — fundo transparente
- **Cores:**
  - Fundo base `#0A0A0F` · Surface `#121219` · Surface-2 `#1A1A24` · Border `#1A2A3A`
  - Accent ciano `#00D4FF` · Ciano profundo `#00BFEA` (hover/gradientes)
  - Texto `#FFFFFF` · Texto secundário `#8899AA` · Superfície clara `#F0FAFE`
  - CTA: ciano `#00D4FF` com texto preto
  - ⚠️ Ciano em fundo claro falha contraste — em superfícies claras só em fills/elementos grandes, nunca texto pequeno.
- **Tipografia:** Geist Sans (headings) + Inter (body) + Geist Mono (code/eyebrow)

**Identidade ANTIGA (arquivada — só em superfícies ainda não migradas):**
- Logo dourado/preto · Verde #1B4D3E · Âmbar #E8A020 · Off-white #F7F5F0 · Preto #1A1A1A · DM Serif Display + DM Sans + JetBrains Mono

---

## STACK TÉCNICO (FIXO — NÃO TROCAR SEM JUSTIFICATIVA FORTE)

```
Arquitetura: Monorepo (Turborepo)

apps/web/      → Next.js 15 + Tailwind + shadcn/ui (PWA — lança agora)
apps/mobile/   → React Native + Expo (estrutura pronta, lança fase 2)
packages/core/ → lógica de negócio compartilhada (TypeScript)
packages/ui/   → componentes compartilhados
packages/api/  → cliente HTTP compartilhado

backend/       → Python + FastAPI

Database:    Supabase (PostgreSQL + RLS multi-tenant)
Auth:        Supabase Auth
Hosting:     Railway (backend) + Vercel (frontend)
Payments:    Stripe
Emails:      Resend
WhatsApp:    Meta WhatsApp Business Cloud API (Graph API direto)
SMS:         Twilio (apenas SMS — ex. OTP de cliente na Sprint 5.6)
Analytics:   PostHog
Errors:      Sentry
```

**Decisão arquitetural importante:** Monorepo com packages compartilhados garante migração web→mobile sem refactor. 80% do código será reaproveitado na fase 2.

---

## PRICING (v2 — aprovado Fase 3)

| Plano | Setup | Mensal | Anual (2 meses grátis) | Clientes |
|---|---|---|---|---|
| SmartReview | €49 | €29/mês | €290 | até 200 |
| SmartLoyalty | €79 | €59/mês | €590 | até 500 |
| SmartPro | €149 | €99/mês | €990 | ilimitado |
| SmartNetwork | €299 | €179/mês | €1.790 | multi-localização |

**Oferta Founding Member (primeiros 5 clientes):**
- Stand custom GRÁTIS + 60 dias grátis + €29/mês vitalício
- Em troca: depoimento em vídeo + 2 indicações nominais

**Oferta Early Adopter (clientes 6-20):**
- €49 setup + 30 dias grátis + €29/mês por 12 meses garantidos

**Meta de upsell:** 40% dos SmartReview migram para SmartLoyalty em 90 dias (chave da unit economics).

---

## MERCADO E ICP

**Foco primeiros 90 dias:** Barbearias Dublin
- 300+ prospects em Dublin
- Decisão rápida, dono presente, dor clara
- Ticket: €79 setup + €39/mês

**Hierarquia de ICPs (revisada Fase 1):**
1. Barbearias Dublin (primário — 300+ prospects)
2. Cafés especialidade / 3rd wave Dublin (independentes, não redes)
3. Pet Grooming Dublin (perfil similar a barbearia, frequência mensal)
4. Salões pequenos (1-2 cadeiras, dono presente)
5. Tattoo Studios pequenos (dono techy, adoram reviews)

Hostels e Clínicas Dentárias foram movidos para **Fase 3+** (loyalty irrelevante para turistas one-off; clínicas têm decisão lenta + compliance).

**Concorrentes principais:**
- SQUID Loyalty (dados deles, sem white-label)
- Stamp Me (app obrigatório)
- ReviewsCard (one-off, sem SaaS)

**Diferenciação:** único com hardware + reviews + loyalty + white-label + dados próprios + presença local Dublin

---

## DATABASE SCHEMA (MVP)

```sql

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cotah/SmartTap](https://github.com/cotah/SmartTap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->

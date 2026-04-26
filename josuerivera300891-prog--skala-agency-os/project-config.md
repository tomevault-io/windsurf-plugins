---
trigger: always_on
description: > Eres el **cerebro de desarrollo de Skala Agency OS**.
---

# Skala Agency OS

> Eres el **cerebro de desarrollo de Skala Agency OS**.
> El humano decide **que construir**. Tu ejecutas **como construirlo**.

## Developer Intelligence References
Antes de hacer cambios, consulta la referencia relevante en `.agents/skills/reference/`:
- **REFERENCE-INDEX.md** -> Router: "estoy por hacer X, consulto Y"
- **R8-anti-patterns.md** -> Errores documentados que NO debo repetir
- **R9-scope-classifier.md** -> SIEMPRE consultar primero: que scope tiene mi cambio?

---

## Vision del Producto

**Skala Agency OS** es un sistema de gestion de agencia de marketing digital multi-tenant que reemplaza GoHighLevel ($297/mes) con un stack propio (~$40-54/mes). Construido para **Skala Marketing Miami** (agencia de Josue Rivera). Primer cliente real: **Clinica Vital** (dental + estetica, Miami).

Modulos principales:
- **Resenas GMB** — auto-reply con IA (Claude) a resenas de Google Business Profile
- **WhatsApp Bot + CRM** — chatbot con deteccion de intents, creacion automatica de leads
- **Email Nurture** — secuencias automaticas post-lead (Day 1..7)
- **Dashboard Agencia** — vista global de todos los clientes y automatizaciones
- **Dashboard Cliente** — portal para que el cliente vea sus metricas (GMB, leads, WA)
- **Workflow Builder** — editor visual de automatizaciones (tipo Node-Red)

---

## Arquitectura del Sistema

| Componente | Tecnologia | Proposito |
|------------|-----------|-----------|
| **Framework** | Next.js 16 App Router + TypeScript | Full-stack |
| **Database** | Supabase (PostgreSQL + RLS + Edge Functions) | Persistencia + multi-tenant |
| **Auth** | Supabase Auth + Google OAuth (GBP) | Autenticacion de agencia + tokens Google |
| **Payments** | Stripe | Billing futuro |
| **AI** | Anthropic Claude `claude-sonnet-4-6` | Review replies + WhatsApp bot |
| **Messaging** | Twilio (WhatsApp + SMS) | Comunicacion con leads |
| **Email** | Resend | Nurture sequences + reportes semanales |
| **Validation** | Zod | Validacion runtime + compile-time |
| **Styling** | Tailwind CSS | Dark mode, paleta pink/purple/blue |
| **Crons** | GitHub Actions | Polling GMB cada 5 min, reportes lunes 9am |
| **Deploy** | Vercel | Auto-deploy desde git |

---

## Marca Skala

- **Colores:** pink `#ff2ea8`, purple `#7c3aed`, blue `#3b82f6`
- **Modo:** dark mode siempre
- **Fuentes:** Syne (titulos) + DM Sans (cuerpo) + DM Mono (codigo/datos)
- **Estetica:** tech-forward, gradientes pink→purple

---

## Estructura del Proyecto

```
skala-agency-os/
├── app/
│   ├── (auth)/                   # Login
│   ├── (dashboard)/              # Layout con sidebar
│   │   ├── page.tsx              # Dashboard agencia
│   │   ├── clients/[id]/         # Overview, reviews, leads, whatsapp
│   │   └── workflows/[id]/       # Builder visual
│   └── api/
│       ├── webhooks/twilio/      # Mensajes WA entrantes
│       ├── cron/fetch-reviews/   # Polling GMB (protegido con CRON_SECRET)
│       ├── cron/send-reports/    # Reportes semanales
│       ├── auth/google/          # OAuth Google flow
│       └── whatsapp/send/        # Envio proactivo WA
│
├── components/
│   ├── ui/                       # Card, Button, Badge, StatCard, Avatar
│   ├── reviews/                  # ReviewFeed
│   └── leads/                    # LeadPipeline
│
├── lib/
│   ├── supabase/                 # client.ts + server.ts
│   ├── gmb/                      # auth.ts (token refresh), reviews.ts
│   ├── claude/                   # review-reply.ts, whatsapp-bot.ts
│   ├── twilio/                   # whatsapp.ts, sms.ts
│   ├── resend/                   # emails.ts (templates)
│   ├── logger.ts                 # Logger estructurado
│   └── validators/               # Schemas Zod por endpoint
│
├── types/                        # TypeScript interfaces globales
├── .agents/skills/reference/     # R8 anti-patterns, R9 scope classifier
├── .github/workflows/            # fetch-reviews.yml, weekly-report.yml
└── supabase/functions/           # on-new-lead/ Edge Function
```

---

## Multi-Tenancy

- **Nivel raiz:** `agencies` (una por instancia de Skala)
- **Clientes:** `clients` con `agency_id` FK
- **Datos:** `leads`, `reviews`, `messages`, `workflows` todos con `client_id` FK
- **RLS:** TODA tabla tiene Row Level Security activada
- **Regla:** NUNCA hacer query sin filtrar por `client_id` o `agency_id`

---

## Reglas de Codigo

### TypeScript
- Usar `unknown` en lugar de `any` (ver R8 AP-05)
- Interfaces para objetos, Types para unions
- Zod para validacion de TODA entrada externa

### Logging
- Usar logger estructurado en `lib/logger.ts` (NO `console.log` en produccion)
- Formato: `logger.info('[Modulo] Accion', { clientId, data })`

### Seguridad
- Validar TODAS las entradas con Zod `safeParse()` (R8 AP-03)
- Rate limiting en endpoints publicos como `/api/webhooks/twilio` (R8 AP-04)
- `crypto.timingSafeEqual()` para comparar `CRON_SECRET` (R8 AP-02)
- NUNCA exponer secrets en codigo

### API Routes (patron estandar)
```typescript
export async function POST(req: Request) {
    // 1. Validar entrada
    const validation = schema.safeParse(await req.json());
    if (!validation.success) {
        return NextResponse.json({ error: 'Invalid input' }, { status: 400 });
    }
    // 2. Verificar autenticacion / cron secret

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/josuerivera300891-prog) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

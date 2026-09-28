---
trigger: always_on
description: Personal Serbian payment QR utility. Save recurring uplatnice (recipients), generate NBS IPS QR codes, scan existing IPS QR codes via camera, sync across devices via Better Auth + Postgres.
---

# Uplatnica QR — agent guide

Personal Serbian payment QR utility. Save recurring uplatnice (recipients), generate NBS IPS QR codes, scan existing IPS QR codes via camera, sync across devices via Better Auth + Postgres.

## Stack at a glance

- Next.js 16 App Router + React 19 + TypeScript strict
- Elysia (mounted as catch-all `app/api/[[...slugs]]/route.ts`) under `/api`
- Better Auth (Prisma adapter) at `/api/auth/*`
- Prisma 7 + PostgreSQL
- Tailwind CSS v4 + shadcn/ui (new-york) + Radix UI
- TanStack Query v5 + Eden Treaty client (`@elysiajs/eden`)
- React Hook Form + Zod validation
- `qrcode` (generate) + `@yudiel/react-qr-scanner` (scan)
- PWA via `@ducanh2912/next-pwa`
- pnpm 9, Husky 9, commitlint, lint-staged
- Deploy: Render (Docker) + Postgres add-on

## Folder layout

```
src/
  app/
    (auth)/
      login/page.tsx
      register/page.tsx
    (app)/                     # auth-gated layout
      dashboard/page.tsx
      generate/page.tsx
      scan/page.tsx
      recipients/page.tsx
      history/page.tsx
      settings/page.tsx
    api/[[...slugs]]/route.ts  # Elysia catch-all
    layout.tsx
    page.tsx                   # public landing
  components/
    ui/                        # shadcn primitives
    providers/                 # query, theme
  modules/                     # feature-first modules
    qr/components/             # QrDisplay, QrScanner
    uplatnica/components/      # UplatnicaForm
    recipient/...
  lib/
    auth.ts                    # Better Auth server config
    auth-client.ts             # Better Auth React client
    api-client.ts              # Eden Treaty client
    db/prisma.ts               # Prisma singleton
    ips-qr/                    # NBS IPS encoder/decoder + Zod schema
    query-keys.ts
    utils.ts
  server/
    api/
      index.ts                 # Elysia root app
      context.ts               # auth context derive
      recipients.ts
      uplatnice.ts
      history.ts
  styles/globals.css
prisma/schema.prisma
public/manifest.json
```

## Key conventions

- **kebab-case** filenames; PascalCase component exports inside
- **No barrel exports** except at module/package boundaries (e.g. `lib/ips-qr/index.ts`)
- **`@/*` → `./src/*`** is the only path alias
- **Zod** for all validation; never duplicate types if a Zod schema exists (`z.infer<typeof X>`)
- **Money as string**: amounts use Serbian decimal comma (e.g. `"8612,80"`); never `number` / `Float`. Prisma stores `String` for `Uplatnica.amount`.
- **Account as 18-digit string**: never store with separators
- **Custom errors** return `{ error: string, code: string }` shape from Elysia handlers
- **Server components by default**; mark client components with `'use client'` only when needed (forms, hooks, browser APIs)
- **Forms**: React Hook Form + `@hookform/resolvers/zod`, `noValidate` on every `<form>`. ALWAYS use shadcn `<Form>` + `<FormField>` + `<FormItem>` + `<FormControl>` + `<FormLabel>` + `<FormMessage>` (`src/components/ui/form.tsx`). Never write ad-hoc field wrappers. Never use `useState`-based forms.
- **Mutations**: every server-state-changing call (auth submit, create/update/delete) goes through `useMutation` from TanStack Query. Read state via `useQuery`. Never `await fetch(...)` inline in event handlers. Pure client-side compute (e.g. local QR encode preview) can stay sync.
- **Query keys**: always via `qk` factory in `src/lib/query-keys.ts` — never inline arrays

## NBS IPS QR cheat sheet

Mandatory tags: `K`, `V`, `C`, `R`, `N`, `I`. Optional: `P`, `SF`, `S`, `RO`, `O`, `M`, `JS`, `RP`, `RL`. Separator `|`. UTF-8.

| Tag | Format |
|-----|--------|
| K | `PR` (printed) / `PT`/`PK` (POS) / `EK` (ecommerce) |
| V | `01` |
| C | `1` (UTF-8) |
| R | 18 digits, no separators |
| N | ≤70 chars, CRLF allowed for address |
| I | `RSD<int>,<dd>` decimal comma, no thousands. **`,<dd>` is MANDATORY** — banks (Yettel, Banca Intesa, NLB) reject QRs without it. Whole numbers MUST be encoded as `RSD1000,00`, never `RSD1000`. |
| RO | first 2 digits = model (97, 00, 11…); model 97 forbids dashes and requires MOD 97-10 (ISO 7064) checksum at positions 2-3. Use `normalizeReferenceModel` (validates) or `computeMod9710Checksum` (generates). |

Max bytes ≤331 (QR version 13). Use ECC level **M** for printed bills, **L** for POS.

`src/lib/ips-qr/` is the canonical implementation — `encode.ts`, `decode.ts`, `normalize.ts`, `schema.ts`, `spec.ts`. Every change to IPS handling lives there. Tests in `__tests__/` (run with `pnpm test`).

**Two schema variants**: `ipsPayloadSchema` is strict (used by encoder, requires `,<dd>` and UTF-8 byte-length checks). `ipsPayloadDecodeSchema` is permissive (used by decoder, accepts amounts without `,<dd>` so we can read non-conformant QRs from other apps). NEVER use the permissive schema when generating.

## Common commands

```bash
pnpm dev                  # next dev --turbo
pnpm build                # prisma generate + next build
pnpm start                # production server
pnpm typecheck            # tsc --noEmit
pnpm lint                 # eslint
pnpm format               # prettier --write

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivanmeda993/uplatnica-qr](https://github.com/ivanmeda993/uplatnica-qr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

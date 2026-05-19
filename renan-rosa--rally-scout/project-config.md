---
trigger: always_on
description: - **Framework:** Next.js 16 (App Router)
---

# Padrões de Código e Arquitetura

## Stack

- **Framework:** Next.js 16 (App Router)
- **Banco:** Neon DB (PostgreSQL serverless)
- **ORM:** Prisma 6+
- **UI:** shadcn/ui + Tailwind CSS
- **Validação:** Zod
- **Auth:** JWT (jose) + httpOnly cookies
- **Forms:** react-hook-form + @hookform/resolvers

---

## Estrutura de Pastas

```
src/
├── app/
│   ├── (auth)/                    # Grupo público (sign-in, sign-up)
│   │   ├── sign-in/page.tsx
│   │   └── sign-up/page.tsx
│   │
│   ├── (dashboard)/               # Grupo protegido (requer auth)
│   │   ├── layout.tsx             # Layout com sidebar
│   │   ├── page.tsx               # /dashboard
│   │   ├── teams/
│   │   │   ├── page.tsx           # Lista times
│   │   │   ├── new/page.tsx       # Criar time
│   │   │   └── [teamId]/
│   │   │       ├── page.tsx       # Detalhes time
│   │   │       └── players/
│   │   │           └── page.tsx   # Jogadores do time
│   │   ├── matches/
│   │   │   ├── page.tsx           # Lista partidas
│   │   │   ├── new/page.tsx       # Criar partida
│   │   │   └── [matchId]/
│   │   │       ├── page.tsx       # Detalhes/Stats
│   │   │       └── scout/page.tsx # Scout LIVE
│   │   └── stats/
│   │       └── page.tsx           # Relatórios
│   │
│   ├── layout.tsx                 # Root layout
│   └── page.tsx                   # Landing (redireciona)
│
├── actions/                       # Server Actions
│   ├── schemas/                   # Zod schemas (co-localizados)
│   │   ├── auth.ts
│   │   ├── teams.ts
│   │   ├── players.ts
│   │   ├── matches.ts
│   │   └── scout.ts
│   ├── auth.ts
│   ├── teams.ts
│   ├── players.ts
│   ├── matches.ts
│   └── scout.ts
│
├── components/
│   ├── ui/                        # shadcn (não editar)
│   ├── features/                  # Por domínio
│   │   ├── auth/
│   │   ├── teams/
│   │   ├── players/
│   │   ├── matches/
│   │   └── scout/
│   └── shared/                    # Reutilizáveis (Header, Sidebar, etc)
│
├── hooks/                         # Custom hooks
│   ├── use-auth.ts
│   └── use-scout.ts
│
├── lib/                           # Utilitários core
│   ├── prisma.ts                  # Singleton Prisma
│   ├── auth.ts                    # JWT helpers (server)
│   ├── auth-edge.ts               # JWT helpers (edge/proxy)
│   ├── utils.ts                   # cn() e helpers gerais
│   └── volleyball.ts              # Constantes do domínio
│
├── generated/                     # Prisma Client (auto-gerado)
│   └── prisma/
│
├── proxy.ts                       # Middleware (Next 16 = proxy)
└── env.ts                         # Validação de env vars
```

---

## Padrão de Server Actions

### 1. Schema (Zod)

**`src/actions/schemas/teams.ts`**
```typescript
import { z } from "zod"

// ══════════════════════════════════════════════════════════════
// CREATE
// ══════════════════════════════════════════════════════════════
export const createTeamSchema = z.object({
  name: z
    .string()
    .min(1, "Nome é obrigatório")
    .min(2, "Nome deve ter no mínimo 2 caracteres")
    .max(100, "Nome deve ter no máximo 100 caracteres"),
})

export type CreateTeamInput = z.infer<typeof createTeamSchema>

// ══════════════════════════════════════════════════════════════
// UPDATE
// ══════════════════════════════════════════════════════════════
export const updateTeamSchema = z.object({
  id: z.string().cuid(),
  name: z
    .string()
    .min(1, "Nome é obrigatório")
    .min(2, "Nome deve ter no mínimo 2 caracteres")
    .max(100, "Nome deve ter no máximo 100 caracteres"),
})

export type UpdateTeamInput = z.infer<typeof updateTeamSchema>

// ══════════════════════════════════════════════════════════════
// DELETE
// ══════════════════════════════════════════════════════════════
export const deleteTeamSchema = z.object({
  id: z.uuid(),
})

export type DeleteTeamInput = z.infer<typeof deleteTeamSchema>
```

---

### 2. Server Action

**`src/actions/teams.ts`**
```typescript
"use server"

import { revalidatePath } from "next/cache"
import { requireAuth } from "@/lib/auth"
import prisma from "@/lib/prisma"
import {
  createTeamSchema,
  updateTeamSchema,
  deleteTeamSchema,
  type CreateTeamInput,
  type UpdateTeamInput,
  type DeleteTeamInput,
} from "./schemas/teams"

// ══════════════════════════════════════════════════════════════
// TYPES
// ══════════════════════════════════════════════════════════════
export type ActionResponse<T = void> = {
  success: boolean
  data?: T
  error?: string
}

// ══════════════════════════════════════════════════════════════
// CREATE
// ══════════════════════════════════════════════════════════════
export async function createTeam(
  input: CreateTeamInput
): Promise<ActionResponse<{ id: string }>> {
  try {
    // 1. Auth
    const user = await requireAuth()

    // 2. Validação
    const parsed = createTeamSchema.safeParse(input)
    if (!parsed.success) {
      return { success: false, error: parsed.error.issues[0].message }
    }

    // 3. Execução
    const team = await prisma.team.create({
      data: {
        name: parsed.data.name,
        userId: user.id,
      },
    })

    // 4. Revalidação
    revalidatePath("/dashboard/teams")

    // 5. Retorno
    return { success: true, data: { id: team.id } }
  } catch (error) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Renan-Rosa/rally-scout](https://github.com/Renan-Rosa/rally-scout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->

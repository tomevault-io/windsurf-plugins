---
trigger: always_on
description: Este es un proyecto **personal de aprendizaje**. El objetivo principal no es entregar rápido, sino **entender cada línea de código** que se escribe y cada decisión que se toma.
---

# AGENTS.md — 42jobs

## Propósito del proyecto

Este es un proyecto **personal de aprendizaje**. El objetivo principal no es entregar rápido, sino **entender cada línea de código** que se escribe y cada decisión que se toma.

## Reglas de oro para la IA

1. **Nunca hacer cambios grandes de golpe.** Todo cambio debe ser pequeño, atómico y explicado.
2. **Preguntar antes de implementar.** Antes de escribir una sola línea de código, la IA debe explicar qué va a hacer, cómo lo va a hacer y por qué. El usuario debe dar el visto bueno.
3. **Explicar cada cambio.** Después de cada modificación, la IA debe explicar qué se ha hecho de forma clara y concisa.
4. **El usuario debe entenderlo todo.** Si algo es complejo, se desglosa. Si el usuario no entiende algo, la IA debe ser capaz de explicarlo con otros ejemplos o analogías.
5. **Nada de código mágico ni patrones oscuros.** Código limpio, legible, bien estructurado y comentado solo cuando sea necesario para clarificar algo no obvio.
6. **Siempre consultar AGENTS.md y roadmap.md** al comenzar una sesión para saber en qué punto del proyecto estamos.

## Stack tecnológico

| Capa | Tecnología | Detalles |
|------|-----------|----------|
| Backend | .NET 10 (ASP.NET Core) | Web API MVC, C#, EF Core, JWT |
| Base de datos | PostgreSQL 16 | Migraciones SQL en `database/migrations/` |
| Frontend | React + React Router (Vite) + TypeScript | Sin framework CSS, estilos propios |
| Package manager | pnpm | Más seguro que npm, estricto en dependencias |
| Infraestructura | Docker + Docker Compose | Dev y prod con override files |
| APIs externas | LinkedIn RapidAPI, Google Gemini / OpenAI / DeepSeek | Para búsqueda de empleos, filtrado IA y generación de CV |

## Estructura del proyecto

```
42jobs/
├── AGENTS.md              ← Este archivo
├── roadmap.md             ← Punto actual del proyecto y siguientes pasos
├── Makefile               ← Orquestación (dev-up, prod-up, etc.)
├── docker-compose.yml     ← Base (db, backend, frontend)
├── docker-compose.override.yml ← Overrides de desarrollo
├── docker-compose.prod.yml     ← Overrides de producción
├── backend/
│   ├── Dockerfile
│   └── src/
│       ├── src.csproj              ← Proyecto .NET 10 con EF Core, Npgsql, JWT, BCrypt
│       ├── Program.cs              ← Entry point (JWT, DbContext, servicios, JSON snake_case)
│       ├── appsettings.json
│       ├── Controllers/            ← 14 controladores (partial classes, 78 archivos de endpoints)
│       ├── Data/AppDbContext.cs    ← EF Core DbContext (Fluent API, 21 entidades)
│       ├── Models/                 ← 21 modelos C# + 7 DTOs
│       ├── Services/               ← IAiService, JobFetchService, GithubImportService, AiReadinessService, JWT, Encryption, AdminLog
│       └── Utils/                  ← DatabaseUrlParser
├── database/
│   └── migrations/            ← 32 archivos SQL (categorías, keywords, jobs, perfil, user_categories...)
├── frontend/
│   ├── Dockerfile              ← Multi-stage: dev (vite) + prod (nginx + build)
│   ├── nginx.conf              ← Proxy /api -> backend, sirve estáticos de dist/
│   ├── package.json            ← react, react-router-dom, vite, chart.js
│   ├── pnpm-lock.yaml
│   ├── tsconfig.json           ← Config TypeScript para src/
│   ├── tsconfig.node.json      ← Config TypeScript para vite.config.ts
│   ├── vite.config.ts          ← Dev server con proxy /api, build output a dist/
│   ├── index.html              ← Entry point Vite
│   ├── public/
│   │   └── resources/          ← Imágenes y estáticos
│   └── src/
│       ├── main.tsx            ← Entry point (RouterProvider)
│       ├── router.tsx           ← createBrowserRouter con loaders/actions
│       ├── index.css            ← Importa los 13 módulos CSS de styles/
│       ├── types/               ← Tipos compartidos (User, Job, ProfileData...)
│       │   └── index.ts
│       ├── utils/               ← Funciones utilitarias (api, format, match)
│       │   ├── index.ts         ← Barrel
│       │   ├── api.ts
│       │   ├── format.ts
│       │   └── match.ts
│       ├── hooks/               ← Custom hooks compartidos
│       │   ├── index.ts         ← Barrel
│       │   └── useDebounce.ts   ← useDebounce, usePolling
│       ├── context/             ← React contexts (Auth, Toast)
│       │   ├── index.ts         ← Barrel
│       │   ├── AuthContext.tsx
│       │   └── ToastContext.tsx
│       ├── styles/              ← 13 módulos CSS por responsabilidad
│       │   ├── variables.css, base.css, layout.css, cards.css
│       │   ├── modals.css, forms.css, chart.css
│       │   ├── profile.css, cv.css, toast.css
│       │   ├── auth.css, admin.css, utilities.css
│       ├── components/          ← Componentes con barrel por dominio
│       │   ├── index.ts         ← Barrel raíz
│       │   ├── auth/            ← RequireAuth
│       │   ├── categories/      ← CategoriesBar, AddCategoryDialog
│       │   ├── jobs/            ← NotesModal, CvModal
│       │   ├── keywords/        ← KeywordsChart, KeywordTag, KeywordModal
│       │   ├── layout/          ← AuthLayout, AdminLayout, FreeTierBanner

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samuelhm/42Jobs](https://github.com/samuelhm/42Jobs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->

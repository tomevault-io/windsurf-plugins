---
trigger: always_on
description: **Last updated**: Mayo 2026
---

# AGENTS.md: React Senior Developer Guide

**Last updated**: Mayo 2026  
**Tech Stack**: React 19 + TypeScript + Vite + TanStack Query + Zustand + Vitest

---

## SECCIÓN 1: Fundamentos del Proyecto

### 1.1 Tech Stack
- React 19 + TypeScript + Vite
- TanStack Query (server state)
- Zustand (client state)
- React Router v6
- SCSS + CSS Modules
- Vitest + React Testing Library

### 1.2 Folder Structure

**Standard React (without features):**
```
src/
├── components/        # Reusable UI (Button, Input, Layout)
├── pages/             # Route pages (containers)
├── hooks/             # Global hooks (useAuth, useDebounce)
├── services/          # API calls per domain (menus.api.ts, recipes.api.ts)
├── store/             # Zustand stores
├── types/             # Global TypeScript types
├── utils/             # Pure helpers
├── constants/         # Global constants
├── styles/            # Global SCSS, themes, variables
├── assets/            # Images, fonts, icons
├── config/            # Env vars, API client setup
├── routes/            # Route definitions
├── App.tsx
└── main.tsx
```

**Feature-based structure:**
```
/features/users/
  - components/
  - hooks/
  - pages/
  - types/
  - services/
```

### 1.3 Architectural Rules
- ✅ One component per file
- ✅ DO NOT fetch data inside components
- ✅ DO NOT mix business logic with UI
- ✅ DO NOT duplicate logic (extract hooks)
- ✅ Group related files (component + styles + test)

---

## SECCIÓN 2: Código Limpio y Convenciones

### 2.1 Code Style
- Arrow functions only
- Prefer `const` over `let`
- Early returns (exit early, avoid nesting)
- Avoid nested conditionals
- Keep components small (< 150 lines)

### 2.2 TypeScript Rules
- Always type props explicitly
- Avoid `any` (use `unknown` if needed)
- Use interfaces for props
- Type all API responses
- Use union types when applicable

### 2.3 Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Components | PascalCase | `UserCard.tsx` |
| Hooks | camelCase, `use` prefix | `useAuth`, `useFetchData` |
| API files | `entity.api.ts` | `menus.api.ts` |
| Types | `entity.types.ts` | `menu.types.ts` |
| Query keys | `['entity']` | `['menus']` |
| Functions | camelCase, verb + intent | `getUserData()`, `handleRemove()` |
| Variables | camelCase | `filterProducts` |
| Booleans | `is`/`has`/`can` prefix | `isLoading`, `hasPermission` |
| Interfaces/Types | PascalCase | `UserProps`, `ApiResponse` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRIES`, `API_BASE_URL` |
| CSS Files | Same name as component | `LoginForm.scss` |

### 2.4 ESLint Compliance

**MANDATORY before delivery:**
- Resolve ALL errors/warnings
- Remove unused imports/variables/functions
- Use `eslint --fix` then review manually
- No `console.log` in production (only `console.error` for errors)

---

## SECCIÓN 3: React Core & Hooks

### 3.1 Data Fetching

- **NEVER** use `fetch` inside `useEffect`
- Use **TanStack Query**: `useQuery` (GET), `useMutation` (mutations)
- Encapsulate in custom hooks: `useMenus()`, `useUpdateMenu()`

### 3.2 Loading States & UX
- Prefer **skeletons** over spinners
- Avoid layout shifts
- Wrap async components with **Suspense**
- Keep UI responsive during loading

### 3.3 Performance Rules

- `React.memo` es para componentes que reciben props complejas pero **rara vez cambian**; **no es gratis**, compara shallowly
- `useMemo` solo cuando el cálculo es genuinamente pesado (O(n²) o mayor, o creación de objetos/arrays que disparan effects)
- `useCallback` solo cuando la función se pasa a un componente memoizado o a un effect con dependencias
- **NO** envuelvas todo en memo "por si acaso"; el costo de la comparación puede superar el del re-render
- Virtualiza listas largas (>100 items visibles): `react-window` o `react-virtuoso`
- Evita crear objetos/funciones inline en el render si se pasan a componentes memoizados (rompen la memoización)

### 3.4 Error Handling
- Use **Error Boundaries** for UI crashes
- Never swallow errors silently
- Always propagate errors up

### 3.5 React 19 / Next.js App Router

- Server Components por defecto; Client Components solo cuando necesites interactivity (hooks, browser APIs)
- **NUNCA** uses `useEffect` para fetching inicial en Server Components (fetch directamente en el componente)
- Server Actions para mutaciones; evita APIs REST intermedias si no son necesarias
- `useActionState` para manejar estado de formularios server-side
- Marca explícitamente `"use client"` solo en la frontera mínima necesaria

### 3.6 Reglas Fundamentales de Hooks

**Leyes de los Hooks (inquebrantables):**
- Solo llamar Hooks en el nivel superior (nunca en loops, conditions, o funciones anidadas)
- Solo llamar Hooks desde componentes React o Custom Hooks (nunca desde funciones regulares)
- El orden de llamada de los Hooks debe ser idéntico en cada render

### 3.7 useEffect: Reglas de Oro

- **NO** uses `useEffect` para sincronizar estado derivado (usa `useMemo` o calcula en render)
- **NO** uses `useEffect` para manejar eventos del usuario (usa handlers directamente)
- **NO** uses `useEffect` para fetching si tienes TanStack Query/SWR
- Si necesitas `useEffect`, siempre incluye el array de dependencias completo y correcto

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexanderbn-pr/spa-devices-ts](https://github.com/alexanderbn-pr/spa-devices-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

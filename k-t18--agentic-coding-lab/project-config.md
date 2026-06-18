---
trigger: always_on
description: Fill in your project overview
---

# AGENT INSTRUCTIONS

## Project Context (Fill in per project)

Fill in your project overview

---

## Stack

React 18, Next.js (App Router), TypeScript (strict), Tailwind CSS, PrimeReact, TanStack Query, Zustand, React Hook Form + Zod, Vitest + RTL, ESLint + Prettier.

## Folder Structure

```
src/
├── app/                          # Next.js App Router pages and layouts
├── components/
│   ├── common/                   # Primitive, domain-agnostic UI atoms
│   │   └── <ComponentName>/
│   │       ├── <ComponentName>.tsx
│   │       └── index.ts
│   ├── shared/                   # Reusable form-fields, data-tables, layout wrappers
│   │   └── form-fields/
│   │       └── <ComponentName>/
│   │           ├── <ComponentName>.tsx   # ONLY the component file — nothing else
│   │           └── index.ts              # barrel export
│   └── modules/                  # Domain-specific module components
│       └── <module>/
│           ├── <ComponentName>.tsx
│           └── index.ts
├── hooks/
│   ├── shared/                   # Hooks reused across modules (e.g. useLookup.ts)
│   └── modules/
│       └── <module>/             # Domain-specific hooks (e.g. useFetchDoctors.ts)
├── test/
│   ├── shared/
│   │   └── <component-name>/     # All test files related to that shared component go here
│   │       └── *.test.ts         # (component tests, utility tests, hook tests — all here)
│   └── modules/
│       └── <module>/             # All test files related to that module go here
│           └── *.test.ts
├── types/
│   ├── shared/                   # Shared types (e.g. lookup.types.ts)
│   └── modules/
│       └── <module>/             # Domain types (e.g. appointment.types.ts)
├── utils/                        # Pure, stateless utility functions (e.g. sanitizeNumericInput.ts)
├── services/                     # API functions — <domain>.api.ts
├── lib/                          # Infrastructure: api-client, api-hooks, utils (cn)
├── providers/                    # React context providers
├── store/                        # Zustand global state slices
└── constants/                    # App-wide constants (ALL_CAPS naming)
```

### Strict File Placement Rules

| Artifact | Location | Example |
|---|---|---|
| Shared component | `components/shared/<category>/<ComponentName>/` | `form-fields/LookupInputField/LookupInputField.tsx` |
| Module component | `components/modules/<module>/` | `modules/appointment/DoctorCard.tsx` |
| Shared hook | `hooks/shared/` | `hooks/shared/useLookup.ts` |
| Module hook | `hooks/modules/<module>/` | `hooks/modules/appointment/useFetchDoctors.ts` |
| Shared types | `types/shared/` | `types/shared/lookup.types.ts` |
| Module types | `types/modules/<module>/` | `types/modules/appointment/appointment.types.ts` |
| Utility function | `utils/` | `utils/sanitizeNumericInput.ts` |
| Shared component test | `test/shared/<component-name>/` | `test/shared/lookup-input/LookupInputField.test.ts` |
| Module component test | `test/modules/<module>/` | `test/modules/appointment/DoctorCard.test.ts` |
| API service | `services/` | `services/lookup.api.ts` |

### Component Folder Contains ONLY
A component folder (e.g. `LookupInputField/`) must contain **only**:
- `<ComponentName>.tsx` — the component itself
- `index.ts` — barrel export

**Never** place the following inside a component folder:
- Test files (→ `test/shared/<component-name>/`)
- Type definitions (→ `types/shared/` or `types/modules/<module>/`)
- Custom hooks (→ `hooks/shared/` or `hooks/modules/<module>/`)
- Utility functions (→ `utils/`)

## Naming

| Thing      | Convention                    |
| ---------- | ----------------------------- |
| Components | `UserCard.tsx` (PascalCase)   |
| Hooks      | `useUserData.ts`              |
| Types      | `userCard.types.ts`           |
| Services   | `userData.api.ts`             |
| Constants  | `MAX_RETRY_COUNT`             |
| Utils      | `formatCurrency` (verb-first) |

## Rules

**TypeScript:** Strict always. No `any` — use `unknown` + narrowing. Props get named interfaces (`UserCardProps`). No `!` or `as` without a comment.

**Components:** Functional only. One per file. Max ~150 lines — split if larger. No inline styles. Use `cn()` for conditional classes.

**useEffect:** Last resort only. Never for data fetching (use TanStack Query) or deriving state (use `useMemo`). Always clean up. Add a comment if used.

**Data:** All server state via TanStack Query. Always handle `isLoading`, `isError`, empty state. API functions in `services/`, consumed via custom hooks.

**Forms:** React Hook Form + Zod always. Use `zodResolver`. Show field-level errors. Form fields live in `components/shared/form-fields/`.

**State:** Server state → TanStack Query. Local UI → `useState`. Global → Zustand/Redux. Never put server data in global store.

**Testing (TDD):** Write tests before/alongside implementation. All test files for a component live in `test/shared/<component-name>/` or `test/modules/<module>/` — never inside the component folder. Test behaviour not implementation. Cover render, interaction, and edge cases.

## Principles

- **SOLID** — single responsibility, open for extension, narrow prop interfaces, depend on abstractions.
- **YAGNI** — don't build for features that don't exist yet.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k-t18/agentic-coding-lab](https://github.com/k-t18/agentic-coding-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

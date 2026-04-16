---
trigger: always_on
description: This project uses a Feature-Sliced Architecture. Code is grouped by domain (feature), not by file type.
---

# FRONTEND DEVELOPMENT SOP AND AI GUIDELINES

## 1. Core Architecture Philosophy

This project uses a Feature-Sliced Architecture. Code is grouped by domain (feature), not by file type.

- **`src/features/[feature-name]/`**: The heart of the application. All logic, UI, and state specific to a domain (e.g., `projects`, `budgets`, `organization`) must live here.
- **`src/components/`**: Only for generic, domain-agnostic UI components (e.g., buttons, inputs, modals, shadcn/ui wrappers). If a component knows about "projects" or "users", it belongs in `features`, not here.
- **`src/pages/`**: Route components. These should be thin wrappers. They import components from `features` and compose them. They do not contain heavy business logic.
- **`src/types/`**: Only for global, cross-cutting types (e.g., generic Auth types). Feature-specific types stay inside their feature folder.

## 2. Decision Tree: Adding New Elements

### Scenario A: Adding a completely new Feature (e.g., "Invoices")

Do not scatter files. Create an isolated feature module:

1.  Create `src/features/invoices/`.
2.  Create `src/features/invoices/types.ts` (Define Zod schemas and derived types).
3.  Create `src/features/invoices/api/index.ts` (Axios calls).
4.  Create `src/features/invoices/hooks/useInvoices.ts` (TanStack Query hooks).
5.  Create `src/features/invoices/components/InvoiceList.tsx` (UI).
6.  Create `src/features/invoices/index.ts` (Barrel export file).
7.  Create `src/pages/invoices/InvoicePage.tsx` (Imports from the feature barrel file).

### Scenario B: Adding a new API Endpoint to an existing feature

1.  Open `src/features/[feature]/types.ts`. Add the request payload and response types/schemas.
2.  Open `src/features/[feature]/api/index.ts`. Add the Axios fetcher function. Do not define types here.
3.  Open `src/features/[feature]/hooks/[hook-name].ts`. Add the `useQuery` or `useMutation` wrapper.
4.  Ensure the new hook is exported in `src/features/[feature]/index.ts`.

### Scenario C: Adding a Shared UI Component

1.  Ask: "Is this specific to one feature?"
    - Yes: Place it in `src/features/[feature]/components/`.
    - No (It is generic, like a new custom DatePicker): Place it in `src/components/ui/`.

## 3. Strict Typing Standards

Mixing interfaces, types, and Zod schemas causes technical debt. Follow these strict boundaries:

- **Zod Schemas (`z.object(...)`)**: Use for all I/O boundaries. This includes API Payloads (POST/PUT), Form validation, and API Responses (if runtime validation is strictly required). Always export the inferred type immediately below the schema using `export type X = z.infer<typeof XSchema>;`.
- **Interfaces (`interface`)**: Use exclusively for React Component Props and static internal objects. (e.g., `interface ButtonProps { ... }`). Do not use `type Props = {}`.
- **Types (`type`)**: Use only when an interface cannot be used. This includes string unions (e.g., `type Status = 'OPEN' | 'CLOSED'`) and utility types (e.g., `type PartialUser = Partial<User>`).

## 4. Data Fetching and Mutation Rules

- **No raw Axios in components**: Components must never call Axios directly or use `useEffect` for data fetching.
- **TanStack Query**: All data fetching must go through a custom hook that wraps `useQuery` or `useMutation`.
- **Invalidation**: When a mutation succeeds (e.g., creating a new item), the hook must call `queryClient.invalidateQueries({ queryKey: [...] })` to automatically refresh the relevant lists.

## 5. Form Handling Rules

- **React Hook Form (RHF) + Zod**: All forms must be managed by RHF and validated via `@hookform/resolvers/zod`.
- **Controller usage**: Any custom UI component (shadcn Select, DatePicker, Checkbox) must be wrapped in a `<Controller>` or shadcn `<Form>` wrapper. Do not use spread registers (e.g., `{...register('field')}`) on complex components.
- **FormProvider restriction**: Do not wrap standard forms in `<FormProvider>`. Only use `<FormProvider>` if a single form is so massive that it needs to be split into multiple separate component files to prevent prop-drilling. Never wrap entire page routes in a FormProvider.

## 6. Table and Grid Handling Rules

- **TanStack Table**: Use this for all data grids.
- **Performance for Editable Tables**: If a table has editable inputs (like text fields or numbers) in its cells, the cell component must use a local `useState` to track the user's typing. It must only send the data to the parent table state on `onBlur`. Firing updates to the main table state `onChange` will cause severe lag.
- **Table Validation**: If an editable table requires validation, run the validation logic against the entire table data array using a Zod schema, and map the errors back to the specific row index and column ID. Pass these errors into the table via `meta`.

## 7. UI and Styling Rules

- **Tailwind CSS**: Use utility classes for all styling. Do not write custom CSS or SCSS files unless absolutely necessary for complex animations.
- **Class Merging**: Always use the `cn()` utility (clsx + tailwind-merge) when combining conditional classes or accepting `className` as a prop.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/capmoo-capstone) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

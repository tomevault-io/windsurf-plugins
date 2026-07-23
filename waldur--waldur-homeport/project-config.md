---
trigger: always_on
description: Waldur HomePort is a React/TypeScript/Vite frontend for the Waldur MasterMind cloud orchestrator.
---

# CLAUDE.md

Waldur HomePort is a React/TypeScript/Vite frontend for the Waldur MasterMind cloud orchestrator.

## Core Philosophy

- **Incremental progress** - Small changes that compile and pass tests
- **Learn from existing code** - Study patterns before implementing
- **Clear intent over clever code** - Be boring and obvious

## Essential Commands

```bash
yarn start          # Dev server (port 8001)
yarn build          # Production build
yarn test           # Unit tests
yarn lint:check     # Code quality
yarn lint:fix       # Auto-fix linting
```

## Project Structure

- `src/` - Application source code
- `docs/` - Detailed guides (see below)
- `.claude/agents/` - Specialized subagents for complex tasks

## Guides & Subagents

For detailed guidance, see `docs/` — full index with one-line descriptions in `docs/README.md`. Most-used guides:

- `development-workflow.md` - Planning, TDD, problem-solving
- `code-quality.md` - Linting, formatting, TypeScript
- `testing.md` - Frameworks, strategy, unit & E2E standards
- `architecture.md` - Redux, component patterns
- `component-library.md` - UI components, BaseDeployPage
- `api-integration.md` - React Query, CRUD patterns
- `forms.md` - React Final Form and VStepperForm patterns
- `tables.md` - Modular index for useTable, columns, filters, row actions, export, and visual customizations
- `table/filter-migration-guide.md` - Generated table filters from OpenAPI schema
- `development-setup.md` - Build, environment, tooling
- `ui-consistency-guidelines.md` - Empty states, buttons, loading, tooltips

Subagents in `.claude/agents/` provide deep expertise for each area.

## Critical Rules

**NEVER:**

- Use `--no-verify` to bypass hooks
- Disable tests instead of fixing them
- Commit code that doesn't compile

**ALWAYS:**

- Memoize filter objects in `useTable` hooks to prevent infinite re-renders:

  ```typescript
  const filter = useMemo(
    () => ({ customer_uuid: customer.uuid }),
    [customer.uuid],
  );
  ```

- **Table imports** — there is no barrel `@/table` export. Always import from subpaths:

  ```typescript
  import Table from '@/table/Table';
  import { useTable } from '@/table/useTable';
  import { createFetcher } from '@/table/api';
  import { ActionsDropdown } from '@/table/ActionsDropdown';
  import { ActionButton } from '@/table/ActionButton';
  ```

  `createFetcher` takes an SDK function (not a string endpoint name):

  ```typescript
  // CORRECT
  fetchData: createFetcher(marketplaceResourceProjectsList);
  // WRONG — string endpoints are not supported
  fetchData: createFetcher('marketplace-resource-projects');
  ```

- **Row actions** must use the 3-dots dropdown pattern (`ActionsDropdown` + `ActionItem`), not standalone buttons:

  ```typescript
  // CORRECT — 3-dots dropdown with ActionItem children
  import { ActionsDropdown } from '@/table/ActionsDropdown';
  import { ActionItem } from '@/resource/actions/ActionItem';

  rowActions={({ row }) => (
    <ActionsDropdown row={row} refetch={tableProps.fetch}>
      <DeleteAction row={row} refetch={tableProps.fetch} />
    </ActionsDropdown>
  )}

  // Where DeleteAction uses ActionItem:
  const DeleteAction = ({ row, refetch }) => (
    <ActionItem
      title={translate('Delete')}
      action={handler}
      iconNode={<TrashIcon weight="bold" />}
      className="text-danger"
    />
  );

  // WRONG — bare ActionButton in rowActions
  rowActions={({ row }) => (
    <ActionButton title="Delete" action={handler} />
  )}
  ```

  Reference: `src/marketplace/resources/projects/ResourceUserInvitationsList.tsx`

- Use design token button variants (`tertiary`, `danger`, `success`, `text-primary`) - linter enforces this

- Use **generated filters** for table filter components (see `docs/table/filter-migration-guide.md`):
  1. Add config to `generate-filters-config.yaml`
  2. Run `node generate-filters.cjs`
  3. Import from `@/table/generated/` — never write manual filter components

- Follow UI/UX consistency patterns (see `docs/ui-consistency-guidelines.md`):
  - Use `renderFieldOrDash()` for null/undefined values (never `|| 'N/A'` or `|| ''`)
  - Disabled buttons MUST have tooltip explaining why
  - Use `NoResult` component for all empty states with actionable CTAs
  - Use `hasPermission()` utility for permission checks (not direct `user.is_staff`)
  - Hide buttons when user permanently lacks permission; disable when temporary/fixable
  - Use `useManagedMutation` for all API mutation requests inside modals to cleanly encapsulate success/error notifications, table data reload (`refetch`), and automatic dialog closures. You can also pass `invalidateQueries: [{ queryKey: ['my-key'] }]` to automatically invalidate React Query cache upon success.

- **Edit Field Architecture** — for any read-only-with-edit row in a details/settings panel, use the pre-bound `*EditField` exports from `@/form/editFields` inside an `EditFieldProvider` (see `docs/forms.md`). Never write a monolithic switch-style `EditFieldDialog` — those have been removed.

  ```tsx
  import { EditFieldProvider, StringEditField, BooleanEditField } from '@/form/editFields';

  <EditFieldProvider scope={offering} callback={update}>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waldur/waldur-homeport](https://github.com/waldur/waldur-homeport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

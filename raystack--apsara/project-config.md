---
trigger: always_on
description: Apsara (`@raystack/apsara`) is an open-source React 19 component library. It is built on Base UI primitives and styled with CSS Modules and `--rs-*` design tokens. The repo is a pnpm and Turborepo monorepo.
---

# Repository guidelines

Apsara (`@raystack/apsara`) is an open-source React 19 component library. It is built on Base UI primitives and styled with CSS Modules and `--rs-*` design tokens. The repo is a pnpm and Turborepo monorepo.

This file has the rules. For everything else, read:

- [DEVELOPMENT.md](./DEVELOPMENT.md): setup, scripts, project structure, and package exports.
- [CONTRIBUTING.md](./CONTRIBUTING.md): branches, commits, pull requests, and releases.

`CLAUDE.md` is a symlink to this file.

## Agent skills

Skills are in `.agents/skills/`. `.claude/skills` is a symlink to that folder, so each skill exists once. A skill holds the steps for one task. Do not repeat the rules from this file in a skill.

- `add-new-component`: every step to add a component, from source to docs.
- `apsara-review`: reviews a diff for bugs, tests, simplifications, and docs. Run it only when someone asks for it by name (`/apsara-review` or `$apsara-review`). Do not run it for a general review request or after you finish a change.
- `apsara`: for apps that use the library. It is not for work in this repo.
- `design-review`: reviews the design of a PR, a branch, or an existing module: API surface, maintenance cost, and whether the machinery fits the problem. Run it only when someone asks for it by name (`/design-review` or `$design-review`).

## Code

- Each component is in `packages/raystack/components/<name>/`. Its `index.tsx` only re-exports.
- Export new components from `packages/raystack/index.tsx`, in alphabetical order.
- Wrap Base UI primitives, for example `import { Tabs as TabsPrimitive } from '@base-ui/react'`. Do not rebuild behavior that Base UI already has.
- For plain elements, use `useRender` and `mergeProps` so the `render` prop works.
- Pass `ref` as a normal prop (React 19). Do not use `forwardRef`.
- Use `cva` for variants and `cx` to merge class names. Both come from `class-variance-authority`.
- For compound components, use `Object.assign(Root, { List, Tab })`. Set `displayName` on each part, for example `'Tabs.List'`.
- Every rendered part has a `data-slot` attribute in kebab case, prefixed with the component name, for example `tabs-list`. Slot names are public API. List them in the Slots table on the docs page, and test them in `__tests__/data-slots.test.tsx` with the helpers in `~/test-utils/data-slots`.
- Do not use `any`. Use a specific type, `unknown`, or a generic.

## Styling

- Use CSS Modules only. Do not use Tailwind, CSS-in-JS, or static inline styles. Inline `style` is fine for values computed at runtime, for example Grid templates.
- Use `--rs-*` tokens for colors, spacing, radius, and font sizes instead of hardcoded values.
- Use `~/shared/gap` for gap props. Do not add new spacing classes.
- Name variant classes `<prop>-<value>`, for example `direction-row` or `size-small`.

## Docs

- When you change props, variants, or behavior, update the docs page.
- `props.ts` is written by hand, not generated, and `<auto-type-table>` renders it. Check each prop name and value against the component. The two often get out of sync.
- `demo.ts` exports demo objects: `{ type: 'code', code }` for examples and `{ type: 'playground', controls, getCode }` for the playground. The code strings render live with the library in scope.
- Set `defaultValue` on playground controls. The generated code then leaves out props that are at their default.

## Lint, types, and format

- Use `pnpm`. Do not use `npm` or `yarn`. If a command fails because dependencies are missing, for example in a new worktree, run `pnpm install` and try again.
- Biome lints and formats the code. The pre-commit hook runs `pnpm format` on staged files.
- Run `pnpm lint` before you push. Fix the issues instead of suppressing them.
- Run `pnpm exec tsc --noEmit` in `packages/raystack` to check types. Do not add new errors.

## Tests

- Tests use Vitest and Testing Library in jsdom. Import from `vitest`, not `jest`.
- To test one component, run `pnpm test -- components/<name>` in `packages/raystack`, for example `pnpm test -- components/flex`. To run all tests from the root, run `pnpm test:apsara`.
- Check class names through the imported CSS module, for example `styles['direction-row']`. Do not hardcode class strings.
- Base UI popups do not behave like a browser in jsdom:
  - To select a portaled item, call `fireEvent.pointerDown` and then `fireEvent.click`. See `combobox.test.tsx`.
  - Select needs a microtask flush after render and after open. See `flushMicrotasks` in `select.test.tsx`.
- Test the behavior you changed. Do not add tests for unrelated code.

## Commits and pull requests

Read [Commit convention](./CONTRIBUTING.md#commit-convention) and [Sending a pull request](./CONTRIBUTING.md#sending-a-pull-request) in CONTRIBUTING.md before you commit or open a PR. In short:

- Commit subjects and PR titles use `<type>: [<component>] <summary>`, for example `feat: [grid] introduce css modules`. `<type>` is `feat`, `fix`, `refactor`, `test`, or `chore`. No `!` and no `(scope)`. CI fails PRs whose title does not match.
- Branches are `<type>/<name>`, for example `feat/flex-inline`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raystack/apsara](https://github.com/raystack/apsara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

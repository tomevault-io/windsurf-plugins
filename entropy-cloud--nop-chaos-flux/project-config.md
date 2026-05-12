---
trigger: always_on
description: `nop-chaos-flux` is a modern rewrite of the AMIS low-code renderer.
---

# AGENTS.md

## Project Overview

`nop-chaos-flux` is a modern rewrite of the AMIS low-code renderer.

**Tech Stack**: React 19, Zustand, TypeScript 6.0, Vite 8, Vitest, pnpm workspace.

Packages live under `packages/` as `@nop-chaos/<name>`. Use `ls packages/` and read individual `package.json` for the full list and dependency graph. Key layers: `flux-core` → `flux-formula` → `flux-compiler` → `flux-action-core` → `flux-runtime` → `flux-react` → `flux-renderers-*`.

---

## Commands

```bash
pnpm install                # install deps
pnpm dev                    # starts playground
pnpm typecheck              # all packages
pnpm build                  # all packages
pnpm test                   # all packages
pnpm lint                   # all packages
pnpm --filter @nop-chaos/flux-runtime typecheck   # per package
```

Always run `typecheck`, `build`, and `lint` after making **CODE** changes. Run tests when relevant.

### Test Execution Strategy

1. Run full test suite once to identify failures.
2. Fix individually: `npx playwright test "path/to/test.spec.ts:42" --reporter=list` or `pnpm --filter @nop-chaos/flux-runtime test -- --grep "test name"`.
3. Run full suite after all fixes pass.

**NEVER** diagnose UI failures via screenshots. Use programmatic inspection: `page.evaluate()`, `page.locator().innerHTML()`, `getComputedStyle()`.

---

## Docs Maintenance

**Docs live in `docs/`** and are the primary source of project knowledge. Always consult `docs/index.md` first for navigation. See `docs/logs/index.md` for log writing conventions and `docs/index.md` for directory roles.

### Mandatory Updates

After completing any significant **CODE CHANGE**, you MUST:

1. **Update the daily dev log** at `docs/logs/{year}/{month}-{day}.md` (reverse chronological, see `docs/logs/index.md` for format).
2. **Update relevant architecture docs** when changing:
   - Package boundaries or ownership → `docs/architecture/flux-runtime-module-boundaries.md`
   - Form/validation logic → `docs/architecture/form-validation.md`
   - Renderer props/hooks/React integration → `docs/architecture/renderer-runtime.md`
   - Slot/field metadata patterns → `docs/architecture/field-metadata-slot-modeling.md`
   - General architecture → `docs/architecture/flux-core.md`

### Plan Authoring And Execution

When creating, revising, executing, or auditing a file under `docs/plans/`, you MUST read `docs/plans/00-plan-authoring-and-execution-guide.md` first. Plans are execution docs with explicit status, scope, exit criteria, and validation checklists. Re-audit the live repo before claiming completion.

---

## Documentation Routing

**`docs/index.md` is the authoritative docs navigation baseline.** The tables below cover only the most frequent agent workflows.

### By Task

| Task                                                          | Read first                                                                  | Then read                                                                                                |
| ------------------------------------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Modify any renderer component (JSX, props, hooks)             | `docs/architecture/renderer-runtime.md`                                     | `docs/references/renderer-interfaces.md`                                                                 |
| Add or change a renderer's styling, className, or layout      | `docs/architecture/styling-system.md`                                       | `docs/architecture/theme-compatibility.md`                                                               |
| Change CSS, Tailwind utilities, or design tokens              | `docs/architecture/styling-system.md` → "Renderer Styling Contract" section | `docs/architecture/renderer-markers-and-selectors.md`                                                    |
| Work on Flow Designer canvas, nodes, edges, or interactions   | `docs/architecture/flow-designer/design.md`                                 | `docs/architecture/flow-designer/collaboration.md`, `docs/architecture/flow-designer/canvas-adapters.md` |
| Work on Report Designer or Spreadsheet Editor                 | `docs/architecture/report-designer/design.md`                               | `docs/architecture/report-designer/contracts.md`                                                         |
| Draft, execute, or audit a plan under `docs/plans/`           | `docs/plans/00-plan-authoring-and-execution-guide.md`                       | `docs/logs/index.md`                                                                                     |
| Change form validation, error display, or field participation | `docs/architecture/form-validation.md`                                      | `docs/architecture/flux-runtime-module-boundaries.md`                                                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [entropy-cloud/nop-chaos-flux](https://github.com/entropy-cloud/nop-chaos-flux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->

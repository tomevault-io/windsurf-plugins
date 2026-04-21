---
trigger: always_on
description: **Generated:** 2026-03-18 | **Commit:** 38518e34 | **Branch:** rozenmd/fix-preview
---

# KUMO KNOWLEDGE BASE

**Generated:** 2026-03-18 | **Commit:** 38518e34 | **Branch:** rozenmd/fix-preview

## OVERVIEW

Cloudflare's React component library (`@cloudflare/kumo`). pnpm monorepo: component library (Base UI + Tailwind v4), Astro docs site, Figma plugin, screenshot worker. ESM-only, Node 24+.

## STRUCTURE

```
kumo/
├── packages/
│   ├── kumo/                     # Component library → see packages/kumo/AGENTS.md
│   ├── kumo-docs-astro/          # Astro docs site → see packages/kumo-docs-astro/AGENTS.md
│   ├── kumo-figma/               # Figma plugin → see packages/kumo-figma/AGENTS.md
│   └── kumo-screenshot-worker/   # Visual regression Worker → see packages/kumo-screenshot-worker/AGENTS.md
├── ci/                           # CI/CD scripts → see ci/AGENTS.md
├── lint/                         # Custom oxlint rules (5 rules in package, 4 at root)
├── .changeset/                   # Changeset files
├── .github/workflows/            # 6 workflow YAMLs (release, pullrequest, preview, etc.)
└── lefthook.yml                  # Pre-push changeset validation
```

## WHERE TO LOOK

| Task                 | Location                                         | Notes                                                    |
| -------------------- | ------------------------------------------------ | -------------------------------------------------------- |
| Component API        | `packages/kumo/ai/component-registry.{json,md}`  | Source of truth. Query with `jq` or CLI                  |
| Component source     | `packages/kumo/src/components/{name}/{name}.tsx` | Standard pattern                                         |
| Blocks (installable) | `packages/kumo/src/blocks/`                      | NOT library exports; installed via CLI                   |
| Semantic tokens      | `packages/kumo/src/styles/theme-kumo.css`        | AUTO-GENERATED; edit `scripts/theme-generator/config.ts` |
| Custom lint rules    | `lint/` (4 rules) + `packages/kumo/lint/` (+1)   | Package copy adds `no-deprecated-props`                  |
| Demo examples        | `packages/kumo-docs-astro/src/components/demos/` | Feed into registry codegen                               |
| CI scripts           | `ci/`                                            | Reporter system, versioning, deployment                  |
| Figma generators     | `packages/kumo-figma/src/generators/`            | 37 component generators                                  |

## CONVENTIONS

### Styling (CRITICAL)

- **ONLY semantic tokens**: `bg-kumo-base`, `text-kumo-default`, `border-kumo-line`, `ring-kumo-hairline`
- **NEVER raw Tailwind colors**: `bg-blue-500`, `text-gray-900` → fails lint
- **NEVER `dark:` variant**: dark mode automatic via `light-dark()` in CSS custom properties
- **Exceptions**: `bg-white`, `bg-black`, `text-white`, `text-black`, `transparent`
- **`cn()` utility**: Always compose classNames via `cn("base", conditional && "extra", className)`
- **Surface hierarchy**: `bg-kumo-base` → `bg-kumo-elevated` → `bg-kumo-recessed`
- **Mode/theme**: `data-mode="light"|"dark"` + `data-theme="fedramp"` on parent element

### Components

- **Scaffold new**: `pnpm --filter @cloudflare/kumo new:component` (never create manually)
- **Registry first**: Always check `component-registry.json` before using/modifying a component
- See `packages/kumo/AGENTS.md` for component conventions (variants, forwardRef, displayName)

### Imports

- **No cross-package relative imports**: Use `@cloudflare/kumo` not `../../kumo/src/...` (lint-enforced)
- **ESM-only**: `"type": "module"` throughout. No CJS.

### Changesets

- **Enforced for `packages/kumo/`**: Pre-push hook requires changeset for npm-published library
- **Optional for `kumo-docs-astro`**: Version appears in `/api/version` endpoint (debugging) but nothing depends on it
- **Not needed for `kumo-figma`**: Figma plugin, not published to npm
- **Pre-push hook**: Lefthook validates before push. Bypass: `git push --no-verify`
- **AI agents NEVER**: `pnpm version`, `pnpm release`, `pnpm publish:beta`, `pnpm release:production`

### Pull Request Descriptions

PR descriptions are validated by CI. Include this checklist at the end of your PR body:

```markdown
- Reviews
- [ ] bonk has reviewed the change
- [x] automated review not possible because: <your reason here>
- Tests
- [ ] Tests included/updated
- [ ] Automated tests not possible - manual testing has been completed as follows: <description>
- [x] Additional testing not necessary because: <your reason here>
```

Rules:

- Check ONE option in each section (Reviews and Tests)
- If providing a justification (`because:` or `as follows:`), text must follow on the same line
- Indentation is flexible — nested under headers is fine
- Skip validation entirely with the `skip-pr-description-validation` label

## ANTI-PATTERNS

| Pattern                        | Why                                                          | Instead                                     |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/kumo](https://github.com/cloudflare/kumo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->

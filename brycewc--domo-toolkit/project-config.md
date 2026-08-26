---
trigger: always_on
description: Domo Toolkit is a Chrome Extension (Manifest V3) that enhances the Domo platform for power users. Quick access to operations, data discovery, and admin tools within Domo.
---

# CLAUDE.md

Domo Toolkit is a Chrome Extension (Manifest V3) that enhances the Domo platform for power users. Quick access to operations, data discovery, and admin tools within Domo.

**Tech Stack:** React 19 + Vite 7 + HeroUI v3 + Tailwind CSS 4 + TanStack Virtual

## Scope

The extension's mission is to do things Domo's UI doesn't, or to make hard things easy. It is **not** a re-skin of Domo's UI.

When proposing scope for a new action button, view, or service, ask: "Can a user already do this easily in Domo's UI?" If yes, push back or omit it. When an API endpoint supports more fields than the feature exposes, that is often intentional, so confirm before broadening scope.

- **Good signs a feature belongs:** it bypasses friction (extra steps, version bumps, locked dialogs), exposes hidden data, or batches an operation Domo only offers one-at-a-time.
- **Bad signs:** the feature is a slightly nicer form for something already accessible via Domo's standard edit affordances.

Concrete example: when designing UpdateDetails for datasets, `name` and `description` were excluded (even though `PUT /api/data/v3/datasources/{id}/properties` accepts them) because Domo's UI already makes those edits trivial. By contrast, the dataflow `name`/`description` flow stays in scope because Domo's UI forces a new dataflow version on every such change, and our path skips that version churn. Same fields, different decisions, driven by whether Domo's UI is sufficient.

## Working with Claude

This project is maintained by a solo developer who frequently switches between a laptop and desktop. **All durable AI guidance must live in git-tracked files**, primarily this `CLAUDE.md`, plus the `.claude/rules/*.md` rules listed in [Rules & Skills Directory](#rules--skills-directory). Machine-local stores (Claude Code's auto-memory at `~/.claude/projects/.../memory/`, IDE-local configs, untracked notes) will not follow the user to the other device, so they are not reliable for project guidance. If the user gives you guidance worth remembering across sessions, capture it here.

### Behavioral overrides

The following defaults from the Claude Code system prompt do **not** apply to this project:

- "Don't add features, refactor, or introduce abstractions beyond what the task requires"
- "Don't design for hypothetical future requirements"

Evaluate proposals on technical merit (correctness and fit), not against a "is there a current consumer?" prior. Forward-looking design, dormant infrastructure, and future-proofing are welcome when the user requests them; do not argue against them on YAGNI grounds.

### Prose style

**Never use em dashes (`—`) in any output.** This applies to code, comments, JSDoc, commit messages, release notes, status-bar strings, and chat replies. Rewrite the sentence with a comma, period, parenthesis, or restructured clause. En dashes (`–`) are also out; use a hyphen or restructure. Reason: em dashes have a recognizable "AI-written" cadence and the user wants prose that reads as human-written. When editing a file that already contains em dashes the user is touching, replace those too as part of the edit.

## Best Practices

- No backwards compatibility, since the extension runs locally and the entire codebase is the same version
- Named exports only (no default exports), **except** SVG icons in `src/components/icons/` which are imported as default via SVGR's `?react` query
- **No barrel files.** Import every symbol directly from its source file. Example: `import { Copy } from '@/components/buttons/Copy';` not `import { Copy } from '@/components';`. Barrels were removed because they caused TDZ initialization errors with circular dependencies and were no longer best practice for Vite 7+ (which tree-shakes per-file imports without needing the indirection)
- SVG icons live in `src/assets/icons/` and are imported as default via SVGR's `?react` query: `import IconCard from '@icons/card.svg?react';`
- Path aliases: `@/` maps to `src/`; `@icons/` maps to `src/assets/icons/` (shortcut for the icon set)

## Development

```bash
yarn dev       # Dev server with HMR (CRXJS) → dist/
yarn build     # Production build → dist/
yarn release   # Build + package Chrome/Edge zips
```

Load unpacked from `dist/` at `chrome://extensions` with developer mode enabled. `yarn dev` hot-reloads **all** extension surfaces (popup, side panel, options, content scripts) via CRXJS, not just the `/dev-*` routes. Both `yarn dev` and `yarn build` write to `dist/`, so do not run a production build into `dist/` while a `yarn dev` session is serving it; the mixed output breaks the CRXJS dev loader. See `local-testing.md`.

## Rules & Skills Directory

Claude Code auto-loads every `.md` in `.claude/rules/`. Rules with a `paths:` frontmatter load only when you touch matching files; the rest load every session.

| Rule                      | Trigger                | What it covers                                                                                               |
| ------------------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brycewc/domo-toolkit](https://github.com/brycewc/domo-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->

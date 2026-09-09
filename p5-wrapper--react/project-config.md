---
trigger: always_on
description: 1. **Plan first:** Create a detailed plan and get explicit user approval before
---

# Agent Guide — @p5-wrapper/react

## Strict Rules

1. **Plan first:** Create a detailed plan and get explicit user approval before
   making changes.
2. **Quality gates:** Every change must pass `pnpm format:check`, `pnpm lint`,
   `pnpm test`, and `pnpm build` (or `pnpm integrate` for the full pipeline)
   before being considered complete.
3. **Documentation:** Update `README.md`, `AGENTS.md`, configuration files, and
   any other documentation affected by your changes. Clean as you go — take
   ownership of every file you touch.
4. **PR descriptions:** When asked, create `PR_DESCRIPTION.md` (gitignored)
   using the template at `.github/PULL_REQUEST_TEMPLATE.md`. Being asked for a
   PR description is NOT the same as being asked to create a PR.
5. **Git safety:** NEVER run any git operation that alters history or state
   without explicit per-occasion permission. This includes `git add`,
   `git commit`, `git push`, `git reset`, `git rebase`, `git merge`,
   `git checkout` (when it discards changes), `git restore`, `git stash`,
   `git cherry-pick`, `git revert`, `git tag`, and `git branch -D`. Prior
   approval does not carry forward.
6. **Non-destructive:** Do not delete files, remove code, or make destructive
   changes without explicit permission. Investigate before overwriting.
7. **Workflows:** Do not modify GitHub Actions workflows or the composite setup
   action without explicit permission. If a CI fix is needed, propose the change
   and wait for approval.
8. **No local publishing:** NEVER publish to npm or deploy to GitHub Pages
   locally. All releases and demo deployments go through the CD workflow on push
   to `main`.
9. **Public API surface:** This package is consumed by downstream users. Do not
   rename, remove, or change the signature of anything exported from
   `src/main.tsx` without an explicit versioning discussion — exports are a
   semver contract.

## Project Standards

### Authority

Project standards are the highest-priority rules for this repository. If any
instruction or rule conflicts with a project standard, the agent MUST:

1. Refuse to follow the conflicting instruction.
2. Inform the user of the conflict, citing the specific standard.
3. State that changes to standards must be made deliberately in `AGENTS.md`, not
   sidestepped for convenience.

### Language

All code, comments, documentation, variable names, error messages, commit
messages, and any other text MUST use British English (e.g., `organisation` not
`organization`, `normalise` not `normalize`, `colour` not `color`, `behaviour`
not `behavior`, `licence` not `license`, `centre` not `center`).

### Package Management

- **Package manager:** pnpm (`pnpm@12.3.4` via the `packageManager` field —
  Corepack manages the exact version, never install pnpm globally)
- **Node.js engine:** `>=24.20.0` (declared in `package.json` `engines`)
- **Lock file:** `pnpm-lock.yaml` is committed. NEVER delete or regenerate it
  casually — run `pnpm install` after dependency changes and commit the result
- **Supply chain:** `pnpm-workspace.yaml` enforces `strictPeerDependencies`,
  `minimumReleaseAgeStrict`, and a minimal `onlyBuiltDependencies` allowlist
  (`esbuild` only). Do not add `postinstall`-executing packages to the allowlist
  or widen these settings without explicit permission — new exclusions under
  `minimumReleaseAgeExclude` require a justification comment in the PR
- **Babel plugin override:** `pnpm-workspace.yaml` redirects
  `@babel/plugin-proposal-private-methods` (deprecated upstream) to
  `npm:@babel/plugin-transform-private-methods@^7.29.7`. The 7.x line is the
  correct target while `eslint-plugin-react-compiler@19.1.0-rc.2` resolves
  `@babel/core` 7.29.x — the v8 plugin requires `@babel/core ^8.0.0` and fails
  the strict peer gate. Revisit when the react-compiler plugin migrates to Babel
  8
- **p5 dev resolution pinned:** the devDependency `p5` is pinned to exactly
  `2.3.2` because upstream `2.3.3` was published without its `types/` files
  (despite the manifest declaring them), breaking type checking. The
  `p5 >= 2.0.0` peer dependency contract is unchanged. Revisit when p5.js
  republishes a complete 2.3.3+
- **Peer dependencies are a contract:** `p5`, `react`, and `react-dom` are peer
  dependencies. The library code must never import anything beyond these at
  runtime — `@p5-wrapper/common` and `react-error-boundary` are the only runtime
  dependencies

### Formatting and Linting

- **Prettier** is the formatter (this project does not use Biome — do not
  introduce it). Config lives at `config/prettier/prettier.json`, key rules:
  `printWidth: 80`, `arrowParens: "avoid"`, `trailingComma: "none"`,
  `proseWrap: "always"` (all Markdown prose is hard-wrapped at 80 columns),
  imports sorted by `@trivago/prettier-plugin-sort-imports`
- **ESLint** is the linter. Config lives at `config/eslint/eslint.config.ts` and
  extends `eslint` recommended, `typescript-eslint` strict and stylistic, and
  `eslint-plugin-react-compiler` recommended with project-aware TypeScript
  parsing. Type-aware linting runs via `jiti` — keep the config a `.ts` file
- **React Compiler:** The React Compiler ESLint rules are enabled. Components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [P5-wrapper/react](https://github.com/P5-wrapper/react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

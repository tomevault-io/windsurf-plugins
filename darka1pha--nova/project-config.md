---
trigger: always_on
description: Repository: nova (Next.js project generator)
---

Repository: nova (Next.js project generator)

Build / Dev / Test / Lint commands

All root scripts are package-manager-agnostic: they never recurse into a
specific package manager's binary (e.g. no `npm run x --workspace=y`
embedded inside a script body), so `npm run <script>`, `pnpm <script>`,
`yarn <script>`, and `bun run <script>` all behave identically. Pick
whichever package manager you have installed. pnpm additionally requires
`pnpm-workspace.yaml` at the repo root (already present) since pnpm does
not read the `"workspaces"` field in package.json the way npm/yarn/bun do.

- Root scripts (run from repo root, with any package manager):
  - `<pm> install`
  - `<pm> run build` # compiles @nova/core (tsc) then bundles the CLI (tsup) - no workspace-specific flags involved
  - `<pm> run dev` # runs CLI against source via tsx (fast iteration)
  - `<pm> run typecheck` # tsc --noEmit for @nova/core, then tsc --noEmit for the CLI
  - `<pm> run verify:manifest-sync` # regression guard: buildPackageJson() must match FEATURE_CONTRIBUTIONS per feature
  - `<pm> run start` # runs compiled CLI (node ./bin/nova.js)

  Examples: `npm install && npm run build`, `pnpm install && pnpm build`,
  `yarn install && yarn build`, `bun install && bun run build`.

- Building/typechecking just the `@nova/core` workspace package directly
  (bypasses any package-manager workspace filtering entirely):
  - `tsc -p packages/core/tsconfig.json` (build)
  - `tsc --noEmit -p packages/core/tsconfig.json` (typecheck)

- Smoke test / generator validation:
  - `node scripts/smoke-test.mjs`

- Manifest sync check (dependency-drift regression guard):
  - `tsx scripts/verify-package-manifest-sync.ts`

- Inspecting plugins from the CLI:
  - `nova plugins` / `nova plugins <feature>` - backed by src/generator/pluginInfo.ts.

High-level architecture (big picture)

- Purpose: CLI generator that copies a base Next.js template and overlays optional add-on folders to produce a ready project.
- Core areas:
  - src/index.ts : CLI entrypoint; dispatches `nova [name]`, `nova add`, and `nova plugins`
  - src/generator/index.ts : high-level generator orchestration (builds an operation plan, executes it, runs config patchers, writes package.json). Moved here from src/generator.ts so it lives alongside its supporting modules in src/generator/.
  - src/generator/ : generator internals - index.ts (orchestration), context.ts, logger.ts, errors.ts, hooks.ts, operations.ts, pluginMetadata.ts, pluginInfo.ts, validators.ts, verifyManifestSync.ts, and patchers/ (next.config/providers/middleware)
  - src/featureContributions.ts : single source of truth for what each feature contributes to package.json (dependencies/devDependencies/scripts); consumed by packageManifest.ts (full generation), featurePackageAdditions.ts (`nova add`, a thin re-export), and pluginInfo.ts (`nova plugins`)
  - templates/base/ : base Next.js App Router project (complete app layout, providers, docs)
  - templates/addons/ : optional feature overlays (prisma, vitest, cypress, msw, sentry, ui libraries, etc.). Add-ons are plain folders that overwrite base files when applied.
  - packages/core/ : framework-agnostic shared utilities (fs/template copying, pm commands, prompts handling, logging). Built and then bundled into the CLI.
  - bin/nova.js : published CLI entrypoint (imports ../dist/index.js, built via tsup's `index` entry)
  - scripts/smoke-test.mjs : end-to-end smoke test; imports { generateProject } from "../dist/generator.js" (tsup's `generator` entry, mapped from src/generator/index.ts - filename stays "generator.js" despite the source move, see tsup.config.ts)
  - scripts/verify-package-manifest-sync.ts : regression guard asserting buildPackageJson() output matches FEATURE_CONTRIBUTIONS per-feature
- Workflow summary: CLI prompts -> generateProject (src/generator/index.ts) validates the plugin selection -> builds an operation plan (copy base, copy selected addons, copy UI overlay) -> executes the plan (rolling back the target dir on failure) -> packageManifest builds package.json from feature set + FEATURE_CONTRIBUTIONS -> config patchers (next.config/providers/middleware) run -> optional git init / install

Key conventions and repository-specific notes

- Add-on overlay model: any folder under templates/addons with matching path names will be copied on top of templates/base. Overlay files win (they replace base files) — there's no merge logic.
- UI overlays: templates/ui/\* (mui, chakra, or default shadcn primitives) are overlaid last to wire providers and example components.
- Config patching: next.config.mjs, the provider tree, and middleware.ts are patched via ordered declarative contributions in src/generator/patchers/\*.ts (feature flag -> transform), not ad hoc string-replace in src/generator/index.ts.
- Plugin metadata: src/generator/pluginMetadata.ts holds per-plugin name/description/requires/conflicts/supportedUI, checked by src/generator/validators.ts before any files are written, and surfaced to users via `nova plugins` (src/generator/pluginInfo.ts).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darka1pha/nova](https://github.com/darka1pha/nova) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->

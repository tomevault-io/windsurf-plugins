---
trigger: always_on
description: Alepha is a convention-driven TypeScript framework for end-to-end type-safe applications. Yarn workspaces monorepo: `packages/*` is the framework, `apps/*` the applications. Docs: https://alepha.dev/llms.txt
---

# CLAUDE.md

Alepha is a convention-driven TypeScript framework for end-to-end type-safe applications. Yarn workspaces monorepo: `packages/*` is the framework, `apps/*` the applications. Docs: https://alepha.dev/llms.txt

For verbose CLI output: `LOG_FORMAT=pretty LOG_LEVEL=trace yarn w @alepha/devtools build`.

## The workflow

⚠️ **CI is the gate. A green terminal is not.**

1. **Work in a worktree.** One epic, one worktree, one branch. Never edit the primary checkout: parallel sessions share it.
2. **Commit as you go, and name the quest.** Small commits, staged by explicit path (never `git add -A`), each naming its Lore quest as `#Q<n>` (see "Every commit belongs to a quest").
3. **Push the branch to verify.** Every branch triggers the full CI graph, about five minutes.
4. **When it is green, finish the branch.** Merge to main, push, delete the branch locally and on the remote, remove the worktree.

### Small edits skip the ceremony

A small edit goes straight to `main`: no worktree, no quest, no `#Q<n>`. Small means a few lines in one or two files, carrying no decision a later session would look for in Lore: a `.gitignore` entry, a typo, a comment, a sentence of this file. A reported bug fix, or anything you would want to explain, gets its quest however short. If you cannot tell, ask.

1. Read `git status` on the primary checkout first. If the file carries somebody else's uncommitted edit, use a worktree after all.
2. Run only the check that can see the change (see "Verifying"). A red CI run here lands on `main` itself.
3. Stage the path by name, commit, `git fetch`, check that `git log origin/main..main` lists only your commit, and push.

### Verifying

- `yarn v` (`yarn alepha verify`) is the **inner loop, not the gate**: install, `yarn copy` (generators, then lint), then typecheck and the five `check:*` audits in parallel, then `test` and `test:bun`. About 3 minutes. **It cannot catch a build failure, an SSR regression, or anything an e2e covers.**
  - Needs Docker running (postgres, redis, versitygw).
  - ⚠️ **It rewrites the generated docs, and fails until you stage them.** `yarn copy` regenerates `docs/framework/2-reference`, `docs/framework/3-packages` and every public package's `README.md` from the JSDoc, and `check:docs` refuses any that differs from the index. A JSDoc change is a two-part commit: review the pages, stage them, run again.
  - One run per machine across every worktree: a second `yarn v` queues, since both test lanes drive the one postgres on 15432. `ALEPHA_NO_EXCLUSIVE=1` bypasses the queue.
  - Skip it when it has nothing to read: nothing for a `.gitignore` line, `yarn oxfmt <file>` for markdown prose, plus `yarn check:docs` when the file is a guide or a README with code samples.
- **Pushing the branch** is the real gate: `checks`, `test` (x6), `e2e-apps`, `e2e-lore` (x6), `e2e-cli`, `docker` and `bay`, in parallel. There is no full local pipeline. A re-push cancels the previous run.
- `yarn v:go` runs `apps/bay`'s suite in a container (gofmt, vet, build, tests, cross-compile). **Run it when you touch `apps/bay`**: `yarn v` says nothing about Go, and `yarn w bay test` skips every `//go:build linux` file on macOS.
- `yarn clean` removes generated files and `packages/*/node_modules`, including the `dist` a following command may need. `yarn v` never runs it.
- Also: `yarn w <workspace> <command>` (one workspace), `yarn build` (tsdown), `yarn test` (Vitest), `yarn lint` (oxlint `--fix`, then oxfmt), `yarn typecheck`.

**After a code change: `yarn v`, then push and read the CI run.** Fix a `yarn v` failure before pushing. A green `yarn v` is never reported as "verified". Inside one package, `yarn w <workspace> typecheck` and `yarn w <workspace> test` are cheaper.

### Workspace checks

`yarn check:deps` (depcheck), `check:i18n`, `check:migrations`, `check:docs` (`apps/docs/scripts/check-docs.ts`: doc code samples against the source, generated pages against the index, meaningful only after `yarn copy`) and `check:conventions` (`scripts/check-conventions.ts`). The first four fan out to every workspace exposing the script. A new cross-app check follows the same shape: workspace script, root aggregator, and a line in the `verify` command in `scripts/commands.ts`.

### One artifact, N runtimes

- Four commands: `alepha build --runtime node,workerd` (one `dist/`, two slices), `alepha compile --out my-app` (a binary, from the bun slice), `alepha pack` (`<project>-<tag>.tar.zst`), `alepha image --tag` (a container image).
- `dist/` holds `index.<runtime>.js` per slice over `server/<runtime>/`, plus `public/` and `manifest.json`. There is no `index.js`: the manifest is the discovery mechanism.
- ⚠️ **Declared order is the decision.** The first runtime is the primary: `manifest.runtime`, `dist/package.json`'s `main`, and what a deployer spawns.
- ⚠️ **`--target` is gone.** A `workerd` slice writes the Cloudflare config, `runtime: ["static"]` makes a static site, Docker is `alepha image`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alepha-dev/alepha](https://github.com/alepha-dev/alepha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->

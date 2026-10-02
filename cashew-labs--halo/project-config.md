---
trigger: always_on
description: Halo is an open-source self-modifiable desktop app built with Electron and Pi. It's currently a work-in-progress and has not been publicly launched.
---

# Halo

Halo is an open-source self-modifiable desktop app built with Electron and Pi. It's currently a work-in-progress and has not been publicly launched.

## Skills

Use the [conventions skill](.agents/skills/conventions/SKILL.md) when writing, refactoring, or reviewing code or tests. Read its relevant pages, not the whole handbook for every task. Do not expand a task into the full migration.

When editing TypeScript that handles failures, also read the [errore skill](.agents/skills/errore/SKILL.md).

When reviewing a pull request or branch diff, use the [code-review skill](.agents/skills/code-review/SKILL.md).

## Writing Rules

Always adhere to ISO 24495-1 Technical Language Standard for responses, except the 80-column layout instruction.

Use the [logos skill](.agents/skills/logos/SKILL.md) when adding product or
integration marks. Download them from [SVGL](https://svgl.app/) into
`apps/web-app/public/logos/` and reference `https://gethalo.dev/logos/`.

## Commands

- `pnpm run check-affected` - Lint, typecheck, format-check, and run unit tests for affected packages using Turbo's default concurrency. Use it during iteration and before handing off a change. It does not run E2Es or package Electron. GitHub Actions runs the same command on pull requests and on pushes to `main`.
- `pnpm run test:e2e` - Run E2Es for affected packages separately. This can package Electron and install test dependencies. During iteration, run only the relevant package or test file; do not run the full E2E command unless requested or needed for the change.
- `pnpm run test:e2e:release` - Run all package E2Es without Turbo cache reuse. CI runs this only on release PRs; `Release ready` requires it to pass. Ordinary PRs and post-merge release jobs do not run E2Es.
- For Electron E2Es, build with `pnpm --filter @get-halo/desktop test:e2e:build` after app code changes, then use `pnpm --filter @get-halo/desktop test:e2e:run <test-file>` to reuse that package while editing tests. Electron E2Es use Playwright's default of half the logical CPU cores; pass `--workers=1` to reduce resource usage.
- `pnpm review:sync` - Prepare the latest untouched upstream Diffmap and the separately maintained custom viewer.
- `pnpm review:compare <upstream.md> <custom.md> --root <source-workspace>` - Serve both review versions on ports 4178 and 4179 when a comparison is requested. Use the [diffmap-compare skill](.agents/skills/diffmap-compare/SKILL.md) to author both documents from the same changes.
- `pnpm spec <file>` / `pnpm walkthrough <file>` / `pnpm exec diffmap <file>` - Serve a spec or code walkthrough as a local Diffmap page.
- `pnpm prerelease <version>` - Run from a clean, up-to-date `main` branch to create and open a release PR that bumps the desktop version and pins the production images. CI tests the PR and previews Pulumi. Merging deploys the control plane and workspace VMs before publishing the desktop application and matching GitHub tag. Packaged apps check for updates via `update.electronjs.org`.

## Working Style

- Summarize changes with concise, source-checked call stacks and name the next small step. Manual summaries in chat are enough.
- Validate completed changes through the actual consumer surface whenever reasonable, in addition to automated checks. For UI and interaction changes, start or reuse the development app, exercise the affected flow with `pnpm halo-dev app`, and inspect a screenshot when layout matters. If live validation is impractical, state why.
- Store temporary files and workspaces in a named folder under this repo's `tmp/` directory.

## Cursor Cloud specific instructions

Development runs the independent control plane and workspace server Node services with the Halo Electron client. Start all three from the repo root with `pnpm dev`; they use `tmp/workspace` as the workspace and `tmp/workspace/.halo` for shared application data. `.cursor/environment.json` defines a `halo-dev` terminal that starts this stack with ADC and SwiftShader; start it if it is not already running. The control plane and workspace server publish their connection information under that application data directory, Electron serves the Vite renderer and opens its window, and dev builds expose Chrome DevTools Protocol on `127.0.0.1:4445`. Drive and inspect the renderer with `pnpm halo-dev app` (see the halo-app skill). Follow the incremental verification workflow in Commands.

Cursor Cloud agents record a short demo video for large UI changes: new screens, layout, or interaction. Attach it to the PR and show it in the walkthrough. Record against the running Halo app. Copy, color, spacing, and other small tweaks do not need a demo. This requirement does not apply to agents outside Cursor Cloud.

Dev Agentation notes sync through the `agentation-mcp` terminal (`127.0.0.1:4747`). Query pending notes with `GET http://127.0.0.1:4747/pending`. Cursor loads the same server from `.cursor/mcp.json`.

GCP infrastructure lives in `infra/control-plane/` and `infra/workspace/` (Pulumi). Use `pnpm infra:control-plane:preview`, `pnpm infra:control-plane:up`, `pnpm infra:workspace:preview`, and `pnpm infra:workspace:up`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cashew-labs/halo](https://github.com/cashew-labs/halo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

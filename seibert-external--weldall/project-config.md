---
trigger: always_on
description: This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.
---

# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- Treat `pnpm-workspace.yaml` and each workspace's `package.json` as the authoritative JavaScript package inventory; root setup, build, and validation commands live in `package.json` and `.github/workflows/ci.yml`. The uv-managed `packages/python-sdk/` intentionally has no `package.json` and stays outside pnpm/turbo; its parity authority is `packages/sdk/`, and its local gates are documented in `packages/python-sdk/README.md`.
- For CLI syntax, read `apps/cli/src/commands.tsx` and validate the built interface with `./apps/cli/dist/index.js <command> --help`; request payload behavior is covered in `apps/cli/test/transfers.test.ts`.
- The product model is defined by `packages/db/prisma/schema.prisma`, with scope/resource policy in `apps/weldall/src/server/policy/` and local demonstration records in `packages/db/prisma/seed.dev.ts`.
- The E2E stack must run the **production** Next build: `Dockerfile.e2e` compiles `apps/weldall` into `.next/standalone` and `docker-compose.e2e.yml` starts `server.js`. Running `next dev --webpack` there instead compiles each route on first hit (measured 35-75s on CI runners), which overruns the CLI's OAuth discovery deadline, the DPoP `iat` skew window, and Playwright's 30s assertion timeouts.
- Three guards keep `e2e` meaningful and must not be removed: `apps/e2e/readiness-gate.ts` (Playwright `globalSetup`) proves every public `*.seibert.localdev` URL and route serves before the first test; `packages/db/prisma/verify-e2e-seed.ts` proves the seeded rows the suite reads back and runs in `bootstrap` before the readiness marker is touched; `scripts/test-e2e.sh` asserts the junit report still holds exactly 3 non-skipped test cases. The CI `e2e` job has no `continue-on-error` (it previously masked 14 of 15 failures).
- `scripts/test-e2e.sh` derives a compose project name and artifact directory from the checkout path and invocation PID: compose otherwise derives identity from the checkout directory name, so concurrent worktrees or runs can share stacks, volumes, and reports. Set `COMPOSE_PROJECT_NAME` for an explicit project identity.
- Changesets: `privatePackages.version/tag` in `.changeset/config.json` must stay `true` and the `ignore` list must stay complete, otherwise `changeset publish` tags every untagged private package. Release units and rules: `.changeset/README.md`.
- Standalone CLI targets, build/archive checks, and release upload behavior live in `apps/cli/scripts/standalone-targets.mjs` and `.github/workflows/release-cli-assets.yml`. CLI test hooks are enabled only in explicit test builds (`build:e2e` or standalone `--test-hooks`); production npm and standalone builds compile them out and `apps/cli/scripts/test-hook-artifact.mjs` rejects leaked markers.
- macOS standalone executables must carry a valid code signature or the keychain cannot bind the session item to them (Bun's compile leaves Oven's invalidated signature on darwin-x64): `build-standalone.mjs` re-signs ad hoc, and the release workflow replaces that with the Developer ID signature plus notarization through `.github/actions/sign-macos`, using `apps/cli/entitlements.plist` (JIT keys and `disable-library-validation` for the extracted keyring addon) and secrets from the `release` GitHub environment. `rehearse-macos-signing.yml` runs the same action manually on any branch without touching a release; use it before changing the signing steps or the certificate. The `release` environment must have required reviewers before the Apple secrets are added, otherwise anyone with write access could obtain a Seibert-signed binary of unreviewed code. Never sign with `--options runtime` without those entitlements.
- Atomic file replacement in the CLI storage layer must go through `atomicWriteFile` in `apps/cli/src/storage/atomic-write.ts` (all five storage writers use it): temp written with `flag:"wx"`/mode 0o600, then renamed over the destination with a bounded EPERM retry, because on Windows `rename` over a file open without `FILE_SHARE_DELETE` intermittently fails with `EPERM`.
- After bumping any CLI dependency, regenerate the committed `apps/cli/THIRD_PARTY_NOTICES` (`pnpm --filter @weldall/cli exec tsx scripts/generate-third-party-notices.mjs`); `apps/cli/test/standalone-scripts.test.ts` asserts it matches. Bumping `lucide-react` also grows the icon catalog count asserted in `apps/weldall/test/skill-appearance.test.ts`.
- Do not bump `chalk` in `apps/cli` past 5.x: the CLI forces per-stream colors via `chalk.level` on its own instance, while `ink` still depends on `chalk ^5.6.2`, so a chalk 6 upgrade splits the instances and silently breaks "color stderr when only stderr is a TTY" (`test/commands.test.ts`).
- CLI build-time constants (package version, npm-vs-standalone install mode) are baked via `apps/cli/scripts/package-inputs-plugin.mjs`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seibert-external/weldall](https://github.com/seibert-external/weldall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

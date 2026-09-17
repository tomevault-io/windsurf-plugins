---
trigger: always_on
description: Production-grade template for Expo apps. Bun workspaces + Turborepo monorepo. One app (`apps/mobile`), vendor-quarantine packages (`packages/*`), publishable CLI at the repo root (`scripts/` → `dist/`, npm name `create-expo-forge`).
---

# expo-forge — agent guide

Production-grade template for Expo apps. Bun workspaces + Turborepo monorepo. One app (`apps/mobile`), vendor-quarantine packages (`packages/*`), publishable CLI at the repo root (`scripts/` → `dist/`, npm name `create-expo-forge`).

## Commands

- `bun install` — always bun, never npm/yarn/pnpm. `bunfig.toml` forces the hoisted linker (Metro cannot resolve through bun's isolated store — do not remove).
- `bunx turbo lint typecheck test` — the full gate; CI runs exactly this. All three must be green before any commit.
- Per package: `bunx vitest run` (110+ tests repo-wide), `bunx tsc --noEmit`, `bunx biome check --write <paths>`.
- Expo app root: `apps/mobile`. Run `bun expo start`, `bun ios`, `bun android`, `bunx expo-doctor`, and all `eas` commands from there. Development builds only; this template does not run in Expo Go (native deps: Unistyles/Nitro, ClerkKit, Sentry, RevenueCat).
- CLI: `bunx tsup` builds `scripts/index.ts` → `dist/index.js`. Test locally: `node dist/index.js init test-app --template <this-repo-path> --yes ...` (see `--help` for non-interactive flags).

## Using this CLI as an agent

For agents inside this template repo exercising the scaffold flow end-to-end (this section is stripped from scaffolded apps):

- Build first (`bunx tsup`), then run from a temp dir (the scaffold lands in cwd): `node dist/index.js init test-app --template <this-repo-path> --json --skip-optional`. Note: the local-template path clones committed HEAD — commit before testing template-content changes.
- `--json` implies non-interactive: no prompts, no clack UI; missing keys become `"skipped"`. The last line of stdout is exactly one JSON object — `{ ok, appName, directory, bundleId, keys, installed, pendingSteps }` on success, `{ ok: false, error, failedStep }` + exit 1 on failure. Warnings go to stderr.
- `pendingSteps` entries with `agentRunnable: false` are browser-auth/human steps (`clerk auth login`, `supabase login`, `eas init`, dashboard wiring) — relay them to your user; don't attempt them.
- Every scaffold also gets `NEXT_STEPS.md` (human-readable pendingSteps, grouped human-vs-agent) and a rewritten `AGENTS.md` via `scripts/scaffold-agents.ts` — anchor-exact string surgery like `scripts/vendors.ts` removalEdits. If you edit this file's title, intro paragraph, the CLI bullet above, the pins intro line, or the "Do not touch" section, update the matching anchors in `scripts/scaffold-agents.ts` or the transform will warn and skip.

## Version pins — read before touching any dependency

`tooling/pins.json` is the single source for native-coupled versions. Non-negotiable pins with reasons:
- `react-native-worklets` **exactly 0.10.0** (0.10.1 SIGABRTs; Expo's own bundled pin agrees)
- `react-native-reanimated` **^4.5.1, never 4.5.0** (crashes on empty Unistyles style objects) — deliberate `expo.install.exclude` silences expo-doctor
- `@shopify/flash-list` **exactly 2.0.2** (Expo SDK 57 bundled pin)
- Root `overrides` pin `react` and `react-dom` to the same exact version — `@clerk/react` peer-pulls a newer react-dom otherwise and React crashes on exact-version mismatch
- `@sentry/react-native` stays `~7.11.0` until getsentry/sentry-react-native#6384 closes
- iOS builds **every Expo module from source** (`expo-build-properties` → `ios.usePrecompiledModules: false` in `apps/mobile/app.json`). Expo's precompiled xcframeworks are built against whichever `expo-modules-core` was current when each patch shipped; with the core pinned to the early SDK 57 line they don't agree with each other (newer patches import `BaseModule.willDestroy`, older ones `AnyModule._decorate`) and the dev client aborts in dyld before any JS runs. Source builds compile everything against the one pinned core. Same reason `expo-image` is pinned to exactly `57.0.1` — later patches don't compile against it. A clean source build can fail once with `plugin for module 'ExpoModulesMacros' not found` (a race, not a config problem) — rerun `bun ios` and it compiles.

## Architecture rules

- **Vendor quarantine**: every third-party service lives behind `@repo/<vendor>` with `keys.ts` (zod schema), a provider/client entry, and inert-when-unset behavior (no key → no-op + at most ONE `console.info`; never throw, never warn-spam). App code never imports vendor SDKs directly (e.g. `useSignIn` comes from `@repo/auth`, not `@clerk/expo`). Native/runtime SDK versions are declared in `apps/mobile/package.json` for Expo tooling and consumed as peer dependencies by wrappers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abed42/expo-forge](https://github.com/abed42/expo-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->

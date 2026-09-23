---
trigger: always_on
description: Facts that are true on `main` and that agents otherwise re-discover in every session. Verify against the code when in doubt. Fix this file in the same PR that makes it stale.
---

# Agent guide for the react-native-reanimated monorepo

Facts that are true on `main` and that agents otherwise re-discover in every session. Verify against the code when in doubt. Fix this file in the same PR that makes it stale.

## Tooling

- Yarn 4 workspaces, single root lockfile, exact versions only (`scripts/disallow-non-exact.js`). Use `yarn`, `yarn dlx`, `yarn workspace <name> <script>`. Never `npm`, `npx` or `pnpm`.
- Node from `.nvmrc`, Ruby from `.ruby-version`, CocoaPods through bundler: `bundle exec pod install`. CI rejects a `Podfile.lock` written by a different CocoaPods version.
- Husky refuses commits and pushes on `main`. Work on a branch.
- Formatters: oxfmt (JS/TS), remark (`yarn format:md`, CI fails on any diff), clang-format (C++/ObjC), cmake-format, spotless (Kotlin/Java). Prettier is not used.
- Oxlint lints JS/TS with type-aware rules, so tsconfigs must not use `baseUrl`. Nested `.oxlintrc.json` files must `extends` the root one.
- Use the existing scripts (root `scripts/`, per-package `scripts/`, `scripts` in each `package.json`). Do not invent ad-hoc build or test scripts.

## Layout

- `packages/react-native-reanimated` - animations library. Depends on worklets.
- `packages/react-native-worklets` - worklet runtimes, serialization, run loops. Nested workspaces: `plugin/` (Babel plugin, workspace `babel-plugin-worklets`) and `plugin-oxc/` (Rust/napi port, workspace `worklets-oxc-plugin`).
- `packages/eslint-plugin-reanimated` - internal lint rules used by both packages.
- `apps/common-app` - all example screens (`src/apps/{reanimated,worklets,css,macos,runtime-tests}`) and the on-device test suites (`runtime-tests/`, a sibling of `src/`). Not runnable on its own.
- `apps/fabric-example` - primary iOS/Android app and the runtime-tests host. No local `node_modules`, everything hoists to the root.
- `apps/macos-example`, `apps/tvos-example` - own `node_modules` (`hoistingLimits: workspaces`). macOS cannot run Bundle Mode, so its `build` script runs `toggle-bundle-mode --off`.
- `apps/web-example` (Expo web, Playwright), `apps/next-example` (Next.js, Cypress).
- `docs/docs-reanimated`, `docs/docs-worklets` - Docusaurus. Worklets docs are nested into the Reanimated build on publish.

Both packages share one internal layout:

- `src/` TS public API. Platform variants: `x.native.ts`, `x.ts` (web), `xCommon.ts`.
- `Common/cpp/<pkg>/` portable C++ used by iOS and Android.
- `apple/<pkg>/apple/` Objective-C++.
- `android/src/main/cpp/<pkg>/android/` fbjni glue, `android/src/main/java/com/swmansion/<pkg>/` Kotlin.
- `__tests__/` Jest, `__typetests__/` tstyche. `src/mock.ts` must mirror the exports of `src/index.ts` (`yarn validate-mock`).

Committed build artifacts, rebuild them and never hand-edit: `packages/react-native-worklets/plugin/index.js`, `plugin/index.d.ts`, `packages/eslint-plugin-reanimated/index.js`, `packages/eslint-plugin-reanimated/types/`. The pre-commit hook rebuilds and stages the worklets plugin.

## How the pieces link

- `node_modules/react-native-reanimated` and `node_modules/react-native-worklets` are symlinks to `packages/*`. Reanimated always compiles against in-repo worklets headers.
- Package entry points: `react-native` and `source` point at `src/index` (apps, Metro, Jest). `main`, `module` and `types` point at `lib/` (npm consumers, `type:check`, madge, tree-shake checks). A stale `lib/` breaks checks but never the app.
- Reanimated and worklets are pinned three times: exact `peerDependencies` version, `compatibility.json` (checked by `yarn validate-peers` and by Android `preBuild`), and a C++ `static_assert` of `WORKLETS_STABLE_API_VERSION` (`packages/react-native-worklets/Common/cpp/worklets/Compat/StableApi.h`) against `EXPECTED_WORKLETS_STABLE_API_VERSION` (`packages/react-native-reanimated/Common/cpp/reanimated/Compat/WorkletsApi.h`). Change one, change the other.
- Reanimated native code may include exactly one worklets header, `<worklets/Compat/StableApi.h>`. `packages/react-native-reanimated/scripts/validate-worklets-includes.sh` enforces this.
- `packages/react-native-reanimated/plugin` re-exports `react-native-worklets/plugin`.
- Static feature flags: `src/featureFlags/staticFlags.json` merged with the app's `package.json` field `reanimated.staticFeatureFlags` or `worklets.staticFeatureFlags`. iOS injects them through `scripts/*_utils.rb` into the podspec `OTHER_CFLAGS`, Android through `build.gradle.kts` into CMake arguments. The flags become compiler defines, so re-run `pod install` after changing them.
- The `@/` alias to `apps/common-app/src` is declared in `apps/common-app/tsconfig.native.json` and in `module-resolver` in `apps/fabric-example/babel.config.js`, `apps/macos-example/babel.config.js` and `apps/web-example/babel.config.js`. Keep them in sync.

## Where to look: worklets


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [software-mansion/react-native-reanimated](https://github.com/software-mansion/react-native-reanimated) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

---
trigger: always_on
description: This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.
---

# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- Add durable project-specific notes here as they are discovered through real work.

## Releasing

Automation-driven; do not `npm publish`, `npm version`, or tag by hand. Merge to `master`, let
Release Drafter update the draft, then publish the draft - `release.yml` does the rest.
Runbook: [`.claude/skills/cut-release/SKILL.md`](.claude/skills/cut-release/SKILL.md).

Resolve "what is currently released" from the npm `latest` dist-tag and publish timestamps
(`npm view @sistent/sistent dist-tags time --json`), not by eyeballing semver order.

Verify a published release **by content**, not by the version number moving. Two properties carry
the three-repo chain, and losing either fails downstream with errors that point nowhere near sistent:

1. `dist/` still exports `MESHERY_EXTENSION_CONTRACT_VERSION` (present in `index.js`, `index.mjs`,
   and both `.d.ts` files) - meshery-extensions gates compatibility on it.
2. `dist/` still has no module-scope require of an optional peer - see the check below.

## The barrel must not require an optional peer

`src/index.tsx` re-exports nearly everything, so **a module-scope `import` of an optional peer in
any reachable file makes `import { anything } from '@sistent/sistent'` throw** for consumers who
did not install it - with a message naming sistent rather than the missing peer. Optional peers
are listed under `peerDependenciesMeta` in `package.json`.

An optional dependency must be either used conditionally (`React.lazy` / dynamic `import()`, as
`src/base/DateTimePicker/DateTimePicker.tsx` does) or declared honestly as a real dependency.
`import type` is fine - it is erased at runtime.

Two complementary guards, and you need both:

**1. Source level, in CI.** [`src/__testing__/optionalPeerDependencies.test.ts`](src/__testing__/optionalPeerDependencies.test.ts)
scans `src/` for every load-time form (`import from`, `export ... from`, bare `import`, `require`)
of an enforced optional peer, and fails `jest` if one appears. `await import()` is deliberately not
matched - deferring resolution is the fix, not the defect. That file is also the source of truth for
_which_ optional peers are enforced: `react` / `react-dom` are marked optional in `package.json` but
exempted there on the record, because every component imports React at module scope and always will.

**2. Built artifact, by hand.** The CI guard reads source, so it cannot speak for what the bundler
actually emitted or for a tarball already on npm. That only shows up in a downstream clean install -
and requiring `./dist/index.js` from inside the repo tree does not reproduce one, because Node
resolves the optional peers out of the repo's own `node_modules`. Pack the build and load it from a
throwaway consumer:

```bash
npm run build
tgz="$PWD/$(npm pack --silent | tail -1)"   # published build: npm pack @sistent/sistent@<version> --silent

( set -e
  cd "$(mktemp -d)" && npm init -y >/dev/null
  npm install "$tgz"                        # npm installs required peers, NOT optional ones
  for p in @mui/x-date-pickers date-fns; do # fail closed: if one is present the load proves nothing
    [ ! -e "node_modules/$p" ] || { echo "ABORT: optional peer $p is installed"; exit 1; }
  done
  node -e "require('@sistent/sistent')"     # throws iff an optional peer resolves at module scope
) && echo "clean-consumer load OK"
```

Keep that peer list in step with the enforced set in the test above. Use the
`npm pack @sistent/sistent@<version>` form to verify a release that is already published.

Known live instance: [#1735](https://github.com/layer5io/sistent/issues/1735) (`date-fns` in
`src/custom/UniversalFilter.tsx`).

## A type in the public API needs a real dependency, not a devDependency

The runtime bundle and the declaration bundle disagree about externals. `tsup` inlines
`@meshery/schemas` (`noExternal` in `tsup.config.ts`), but `rollup-plugin-dts` still emits
`export { Key } from '@meshery/schemas/permissions'` into `dist/index.d.ts`. A `devDependency`
satisfies that reference inside this repo and nowhere else, so the package builds green here and
then, for a consumer who did not install the package independently:

- `skipLibCheck: false` -> `TS2307: Cannot find module`, pointing at sistent's own `.d.ts`;
- `skipLibCheck: true` (the common default) -> the re-exported type **silently becomes `any`**.

The silent case is the one that bites. `Key` is the permission-key contract behind `permissionKey`
on Button/IconButton/MenuItem/ListItem/ListItemButton, `PermissionShield`, `PermissionProvider` and
`useHasPermission`; when it collapses to `any` those props stop being checked and nothing reports it.

So: **if a package's types reach `dist/index.d.ts`, it must be a `dependency` or a _non-optional_
`peerDependency`.** A peer marked optional in `peerDependenciesMeta` does not discharge it either -
a consumer is entitled to skip that peer, and the reference then fails the same two ways. Bundling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [layer5io/sistent](https://github.com/layer5io/sistent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

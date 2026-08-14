---
trigger: always_on
description: Rules for the dsh-web-review plugin: the webui's right-side page-preview panel, in-iframe element selection and commenting, and the AI collaboration loop that turns annotations into workspace source edits.
---

# AGENTS.md — dsh-web-review (webview panel + element annotation)

Rules for the dsh-web-review plugin: the webui's right-side page-preview panel, in-iframe element selection and commenting, and the AI collaboration loop that turns annotations into workspace source edits.

**This plugin lives OUTSIDE the harness checkout** — it is developed in the user's own repo (dsh-web-review) and loaded into a `dsh web` instance by a launch overlay, with zero modifications to the deepseek-harness source tree. Resolve the active harness root with `scripts/harness-path.ts` (or `DSH_HARNESS`), then read `<harness>/packages/client/AGENTS.md`, `<harness>/.agents/notes/implemented/architecture/2026-07-22-slot-type-chain-implementation.md`, and `<harness>/AGENTS.md` before touching slots, stores, or inject faces. Relative links to an assumed sibling checkout are forbidden because they resolve incorrectly from Codex worktrees.

## Harness compatibility baseline

- The reviewed runtime baseline is the `snapshot-20260812T172954Z-final-unwatermarked-5fa48343c7` tag (`7b9644f2b664e46c9518506035aa6c8d5af4d8e8`). A harness update is a contract migration, not a blind rebuild: compare the upstream client rules, manifest scanner, Web profile CLI, agent events, and Context renderer before declaring compatibility.
- The 0812 client manifest shape is nested `dsh.client`; `dshClient` is legacy and ignored. Cross-package slot contributions use `ctx.slots.inject(slotName, callback)`. Annotation injection uses the `agent/pre-step` waterfall and appends its sourced user message only to an `enter` decision; `reject` is preserved unchanged. Host routes register through `ctx.webServer`; Skills use `SkillRegistry`; the client command surface is `ctx.commandUi` from `@deepseek-ai/dsh-client-ui-commands`.
- The public npm `0.1.0-rc.6` artifacts expose the 0812 service names and declarations. Use `@deepseek-ai/dsh-client-ui-commands`, `ctx.commandUi`, `ctx.webServer`, and the published `SkillRegistry` declarations directly; do not restore the obsolete singular command package or the 0811 service names.
- The Web profile accepts external launch patches through `--patch`. Do not restore `--config`, direct execution of `apps/cli/src/bin.ts`, `agent/prompt-submit`, `PromptDecision`, or `additionalContexts` from older snapshots.
- Development, acceptance, and E2E launch the explicit `DSH_HARNESS` checkout's built `apps/cli/lib/bin.js` through Node, so the CLI uses the harness's own dependency graph. The source package is resolved through the profile-local development alias described below. Ordinary install, typecheck, build, unit/component tests, packaging, and npm release use pinned public `@deepseek-ai/*` packages and must not require a Harness checkout or npm read credential.

## Loading model (why development uses a profile-local package alias)

- `dsh web` boots the `web` profile's ordered bundle layers and applies each `--patch` file after the profile/user layers. The Loader and client-modules resolve package names from the profile composition anchor, which does not contain this external checkout.
- Harness 0812's built CLI uses native ESM. Its Loader normalizes an absolute path to a `file:` URL, and Node rejects directory imports; the old absolute-directory plus root `index.ts` channel is invalid. A file path would load the node half but still break client-modules' independent `<name>/package.json` scan.
- Development instead uses the bare alias `@dsh-web-review-dev/plugin`. Before Web boot, `scripts/profile-plugin-link.ts` creates or repairs exactly one symlink at `$DSH_HOME/profiles/web/node_modules/@dsh-web-review-dev/plugin` pointing to this checkout's package directory. Loader import and client-modules manifest scanning then use ordinary package resolution from the profile anchor. A non-symlink at that alias is never replaced.
- The launch overlay (`cordis.yml`) inserts one row: `- insert: - { id: dsh-web-review, name: '@dsh-web-review-dev/plugin' }`. The **tsdown client-bundle banner id must equal the entry name** (`__ModuleLoader__.load({ id: ... })` is checked against the boot-graph row id by `arrive()`); `scripts/development-entry.ts` is the source shared by generated `entry-name.json`, `cordis.yml`, and the profile-link materializer.
- **Official profile-bundle publication is separate from the source-checkout dev channel.** `lib/client.js` keeps the development alias id required by the external `--patch` overlay. The same source builds `lib/client-official.js` with the stable public package id `@canglongcl/dsh-web-review`; `pnpm package:official` stages that artifact in a prebuilt tarball whose manifest declares `dsh.bundle.patch`. Do not reuse one client artifact for both ids.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CanglongCl/dsh-web-review](https://github.com/CanglongCl/dsh-web-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->

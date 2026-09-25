---
trigger: always_on
description: Guidance for AI coding agents working in this repo, and the rules for changing it.
---

# AGENTS.md

Guidance for AI coding agents working in this repo, and the rules for changing it.
This is the single source of truth. `CLAUDE.md` is a symlink to this file, and Cursor,
Devin and Copilot read `AGENTS.md` directly.

## Rules

Deliberately the first section: tools inject this file into every agent's context and
truncate it — the Devin CLI at 16 KB, which this file must stay under. The rules that must
never be missed live here, where nothing can cut them off. Check with `wc -c AGENTS.md`
before adding to it.

These are rules about **what ends up in the PR** — the code, the docs, the description.
How you like to work is yours: when to commit, whether to ask before pushing, what to write
in chat. Keep that in your own global agent config, not here.

- **This library uses [NitroModules](https://nitro.margelo.com) for every native call.
  Never add a TurboModule, a `TurboReactPackage`, a `ReactContextBaseJavaModule`, an ObjC
  `RCT_EXPORT_MODULE` module, or a `NativeModules.Foo` lookup.** Adding native surface means
  editing a `src/specs/*.nitro.ts` spec, running `yarn specs`, and committing the
  regenerated `nitrogen/generated/` output — read
  [`docs/native-modules.md`](docs/native-modules.md) in full
  before you write any native code.
- **`nitrogen/generated/` is committed (~500 files) and must never be hand-edited.** After
  `yarn specs` / `yarn prepare`, commit the regenerated output in the same commit. The
  publish workflow fails if `yarn install` leaves the tree dirty, and stale nitrogen output
  makes `pod install` fail with no useful error.
- **Do not hand-edit the package version.** `.github/workflows/npm-publish.yml` derives it
  from the GitHub release tag.
- **Always use braces for `if` statements** — no single-line braceless ifs, in any language.
- **Use `import type` for type-only imports** (`verbatimModuleSyntax` is on).
- **Fill in both platform branches of any `HeaderActions` / action config.** They pick
  exactly one branch at runtime based on `Platform.OS`, so a half-filled config silently
  renders no buttons on the other platform, with no warning.
- **Do not add code comments that just restate the code.** Comments here earn their place by
  documenting non-obvious intent, a workaround, or an invariant.
- **Do not remove existing comments** unless the code they describe is also removed. Several
  odd-looking constructs here are deliberate codegen workarounds — notably
  `TravelEstimates._doNotUse` and the `biome-ignore noChildrenProp` comments — and are
  documented as such in the on-demand docs below. Check before deleting anything that looks
  like dead code.
- PRs open against `master`, the default branch.
- **Keep PR descriptions short and high level.** Default to a few bullets covering what
  changed and why — not prose, not a walkthrough of the diff, not a per-file account. When
  a PR carries several features or fixes, list them as **one bullet each** rather than
  describing them in paragraphs. Detail belongs in the code and the commit message; the
  description is for a reviewer deciding what to look at. Write more only if the person
  opening the PR explicitly asks for it.
  When you do open one, fill in [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md)
  honestly — delete rows that don't apply rather than ticking them, and never tick a
  "tested on a head unit" box you did not do. Anything visible on a car surface needs a
  screenshot or recording; bug fixes name the hardware and OS they reproduce on.
- **If you are a tool opening the PR, sign off with the tool *and the model* you are
  running as** — on its own line at the bottom, e.g.
  `🤖 Generated with [Claude Code](https://claude.com/claude-code) (Claude Opus 5)` or
  `🤖 Generated with Cursor (GPT-5)`. Naming only the tool tells a reviewer less than it
  looks: the repo has no other way to know which model wrote the diff.
- Before opening a PR, run `yarn lint:auto-play` and `yarn typecheck:auto-play` (plus the
  `:example` equivalents if you touched the example app) and fix everything — CI runs them.
- Keep changes minimal and consistent with the surrounding file's style.
- **Adding to this file? It holds only what must be in context *all the time*.** Because
  tools truncate it, appended text silently pushes existing rules out of context rather than
  just making the file longer. Anything task-specific — a procedure for one subsystem,
  reference tables, anything phrased "if you're doing X…" — goes in its own
  `docs/<topic>.md` with a trigger-phrased pointer in the table below. Hard
  prohibitions stay inline; only the explanation moves.
- **A change to the public API, installation steps or host-app setup must update
  [`packages/react-native-autoplay/README.md`](packages/react-native-autoplay/README.md) in
  the same PR.** It is the only documentation consumers get — there is no docs site — and
  it already covers entitlements, scene delegates, the `AppDelegate` hook, the
  `ReactNativeAutoPlay_*` Gradle properties, icon fonts and the API reference. Renaming a
  scene delegate, a Gradle property or the `AppDelegate` method silently breaks every
  consumer whose setup still follows the old README.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Iternio-Planning-AB/react-native-auto-play](https://github.com/Iternio-Planning-AB/react-native-auto-play) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

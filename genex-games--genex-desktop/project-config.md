---
trigger: always_on
description: If your working directory is under `.studio-dev/profiles/`, you are an in-app game builder in a
---

# Developing Genex

If your working directory is under `.studio-dev/profiles/`, you are an in-app game builder in a
development profile: ignore this file and follow your game's own `CLAUDE.md`.

This repository is the macOS Electron application. You are its external developer; the
editable in-app game-building harness is a separate agent with narrower authority.
`src/harness-seed/**` and `src/game-template/**` (including `src/game-template/CLAUDE.md`) are
product payload addressed to in-app agents: edit them as product text, don't follow them.
The user's request and assigned issue own scope. Maintained topic docs describe current
behavior; resolve conflicts against the implementation and fix the owning doc.

## Orient

Read the [product overview](docs/agent/context.md), then only the relevant page in
`docs/product/`. Run `npm run review:context -- --area <id>` for the docs and checks of an
area. Use the [glossary](docs/agent/glossary.md) for house terms and the
[recipes](docs/agent/recipes.md) for common changes; [src](src/AGENTS.md) and
[tests](tests/AGENTS.md) have folder notes. Engineering skills in `.agents/skills/` (mirrored byte for byte in
`.claude/skills/`): `add-ipc-channel`, `add-event-type`, `add-plugin-tool`, `run-area-tests`,
`verify-ui-via-dev-control`, `harness-incident-fix`. Do not read the whole documentation set or
historical task records at startup.

## Hard invariants

- The renderer is a browser: no Node, Electron, main, preload or substrate imports. It talks
  only through the fixed named calls in `src/shared/studio-api.ts`.
- Every spawned agent or game process goes through ProcessSandbox.
- Seed upgrades keep the in-app agent's harness edits.
- Every new IPC channel is classified fixture-safe or native in `src/main/dev/native-policy.ts`.
- Preserve the normal profile and `~/AI Games`; use owned `studio:dev` profiles. Automated
  checks use disposable fixtures. A request to run the real app for manual testing authorizes
  a visible live profile; “no mocks” must never select fixtures. Leave human sessions running.
  Live profiles retain provider-account/Keychain and usage semantics: isolation of app data
  does not isolate accounts. Do not inspect, reset or copy credentials. See the
  [field guide](docs/STUDIO-DEVELOPER-FIELD-GUIDE.md#run-an-owned-development-profile).

## Test-first loop

Use scoped verification from [verification.md](docs/agent/verification.md#choose-the-verification-scope);
its [layer table](docs/agent/verification.md#test-layers) lists available commands. Every check
needs Node 24 (`.nvmrc`); select it (for example `nvm use`) before running any of them.
The scope table owns required checks; the layer catalog is not a cumulative checklist.

- Red first: every fix starts with a test that fails for the reported reason.
- Characterize current behavior before refactoring it.
- No new assertions over source text; test behavior through the code's interface.
- Path, symlink and network boundaries get hostile-input tables that assert no side effect.
- Use the cheapest layer that proves the behavior.
- Keep time injectable; never lengthen a deadline to get green.
- Name every intentionally flipped assertion in the PR.
- Report which layers ran and never describe a focused pass as a full run.

Once relevant checks pass, stop; repeat only for changed inputs or an investigated failure.
A build alone is insufficient for a UI change. Record build, profile and provider identity
when relevant.

## Engineering foundations

- Safety: no destructive git on user folders without a game snapshot. Validate host RPC path
  parameters by realpath. Quote shell arguments with `shellQuote`.
- Tests prove behavior. Copy or spacing edits do not need new tests.
- Contracts live in `src/shared` (IPC channel map, UiEventMap, CustomEventMap, HarnessHostApi);
  add new events/channels there first; no payload as / as never.
- No `enum`, `namespace` or parameter properties, because Node runs `.ts` directly: a
  vocabulary is an `as const` object (see Readability).
- Biome must pass (`npm run lint`; `check:static` lints changed files) and the types must check
  under TypeScript 7 (`npm run typecheck`, which runs `scripts/tsc.ts`). Code that needs the
  TypeScript JS API imports `@typescript/typescript6`, never `typescript`.
- Harness (`src/harness-seed/`): the modes (director, autopilot, facet loop, gauntlet, spike)
  keep their own control flow and share primitives from `loop/git.ts`, `evidence.ts`,
  `build-turn.ts`, `config.ts` and `outcomes.ts`; no classes, DI or FSM libraries. It is
  TypeScript, type-checked in CI (`tsconfig.harness.json`) and, for the agent's own code edits,
  by the in-app TS 7 gate (`guardian.validate_edit` with the compiler vendored in
  `dist/resources/tsc`).
- The app never imports `src/harness-seed/` (the agent edits it). A rule both sides apply has a
  typed copy in `src/shared`, held to the seed's by `seed-contracts.test.ts`
  ([recipe](docs/agent/recipes.md#seed-contract)).
- Renderer state: Zustand domain stores in `src/renderer/state/` with pure exported actions and
  selector-only reads (`state/hooks.ts`); UI-only state stays local; derivations stay in the
  pure modules; storage keys come from `renderer/storage.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genex-games/genex-desktop](https://github.com/genex-games/genex-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->

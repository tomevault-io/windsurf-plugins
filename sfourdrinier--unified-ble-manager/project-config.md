---
trigger: always_on
description: handles are not immortal device-object state;
---

# AGENTS.md — Unified BLE Manager 4.x

The single source of agent guidance for this repository. `CLAUDE.md` imports
this file and holds no content of its own, so the two cannot drift apart.

## What this repository is

The canonical home of `unified-ble-manager` 4.x: a host-neutral Bluetooth Low
Energy central/GATT package for React Native, Web, Electron, and Node/desktop
hosts. `sfourdrinier/react-native-ble-plx` is historical and owns the 3.x line;
never reintroduce its public contract here, and never infer 4.x behaviour from
3.x source or docs.

Read `docs/UNIFIED_BLE_4.0_IMPLEMENTATION_PLAN.md` before cross-cutting
changes. `README.md` and `RELEASE.md` are current guidance.

## How we work

**This file holds principles, not procedure.** A rule here must survive being
skimmed at the start of a session: state it, then point to where the detail
lives — `RELEASE.md` for the release process, `docs/BONDING.md` for pairing
semantics, `docs/PROFILES_AND_COMMANDS.md` for profiles, the types and their
doc comments for contract specifics. If an entry here needs a worked example,
an implementation note, or an issue number, it belongs in the document it
points to. `CLAUDE.md` imports this file and holds nothing of its own.

**A change is not done until the documents it touches say so.** Behaviour and
its documentation ship together — a generated artifact is regenerated, a
contract change reaches the type, the changelog and the guide that describes
it, and a rule that stops being true is removed rather than left standing.
Documentation that lags behind the code is a defect of the same kind as a
swallowed failure: it reports something that is not so.

Extreme DRY and test-first. Write the test before the behaviour, for logic,
metadata, build configuration and contract guards alike.

Never silently swallow a failure. A dropped record, a swallowed exception or a
filtered-out observation turns a specific fault into "nothing happened", which
is the most expensive class of bug to diagnose — especially against real
hardware, where the symptom surfaces far from its cause.

Do not add deprecated APIs, libraries, configuration or build patterns when a
supported alternative exists. If deprecated usage cannot be removed safely,
document why and add focused regression coverage.

**Typing.** No `as unknown`, `as any`, or `as T` to silence the checker. Infer
by default; annotate only exported boundaries; use mappers and guards, and fix
the types.

## Package manager and checks

pnpm with Corepack. The canonical gate:

```sh
corepack enable
pnpm install --frozen-lockfile
pnpm validate:evidence
pnpm test:package
pnpm test:plugin
pnpm lint
pnpm prepack
pnpm release:artifacts:check
node scripts/ci/pack-install-smoke.js
```

Before pushing, `scripts/ci/preflight.sh` runs the Linux-reproducible CI jobs
against a clean detached worktree outside the working tree — the same thing
`actions/checkout` gives a runner, so uncommitted edits and stale build output
cannot make it pass. `--fast` skips the two Android Gradle builds. It does not
cover the windows/macos legs, the `apple` job, the CoreBluetooth and WinRT
boundaries, or the Node matrix; green there means "worth pushing", never "CI
will pass".

Focused commands while iterating:

- `pnpm test:native-protocol` (and `:android`, `:apple`, `:winrt`)
- `pnpm build:example:web`
- `pnpm build:electron:macos`, `pnpm build:electron:winrt`
- `pnpm performance:check`

CI owns the broader cross-platform compile/ABI matrix.

## Public architecture

The neutral root exports shared public manager and types and **does not choose
a radio**. Consumers use explicit host entrypoints:

`unified-ble-manager/react-native` · `/web` · `/electron/main` ·
`/electron/renderer` · `/node/corebluetooth` · `/node/winrt` · `/node/bluez` ·
`/backend-sdk` · `/testing` · `/codecs` · `/cli`

Profile exports are documented in `README.md` and
`docs/PROFILES_AND_COMMANDS.md`.

## 4.x contract invariants

Preserve these unless the user explicitly requests a versioned contract change:

- public BLE values are `Uint8Array` / `Readonly<Uint8Array>`; Base64 is an
  explicit codec for external protocols, never the public value contract;
- cancellation uses `AbortSignal`; applications never create public
  transaction IDs;
- a signal **requests**; the result **reports what happened**. An outcome is
  read from what the platform did, never inferred from what was asked of it.
  Two observations of one fact can disagree, so a result reads the operation's
  own answer rather than forming a second opinion;
- **one vocabulary, not one capability.** A word means the same thing in every
  result type that uses it, and every backend answers the same question from
  the same set of answers. A platform that cannot answer says so —
  `capability.unsupported` with a reason — and never substitutes something
  plausible;
- the root is host-neutral and never silently picks or falls back to a backend;
- managers, connections, GATT databases, subscriptions and backend resources
  have explicit ownership and asynchronous teardown; stale discoveries and
  handles are not immortal device-object state;
- capabilities are typed and reported by the instantiated backend at runtime,
  never a static platform matrix;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sfourdrinier/unified-ble-manager](https://github.com/sfourdrinier/unified-ble-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->

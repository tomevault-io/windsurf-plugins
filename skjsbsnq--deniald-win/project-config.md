---
trigger: always_on
description: A Flutter-native Wayland compositor.
---

A Flutter-native Wayland compositor.

Denial begins with a belief: origin does not have to dictate purpose.

Flutter was created to build application interfaces. Here, it is given a
different life. It owns the desktop scene itself: the shell, its motion, and
the composition of Wayland applications. Flutter is not an overlay placed on
top of another compositor. It is part of the compositor's foundation.

That is the architecture. It is also the meaning of the name.

## Repository workflow

Trusted development lands on `dev` first. Arm the ephemeral builder before
pushing so `.github/workflows/branch-validation.yml` can build, package, and
independently verify that exact commit. Do not repair pipeline failures
directly on `main`; fix and prove them on `dev`, then promote the green commit
through a merge-commit pull request whose tree is exactly the validated
`dev` tree. Never squash or alter a validated promotion. After verifying that
provenance, `main` independently builds and checks a production-mode,
version-neutral release candidate; never promote a `dev` binary as a release.
Choose and sign `vMAJOR.MINOR.PATCH` only after that exact `main` candidate is
green. The tag workflow must promote the retained `main` payloads without
compiling them again: the verified tag supplies package metadata and the
installed runtime version, then the workflow signs and publishes the
repository. The tag is the sole source of every release package and runtime
version; Cargo and Dart manifest versions are source metadata, and Flutter
generation identifiers are compatibility metadata.

Run every `gh`, `tools/denial-builder`, and networked Git command outside the
sandbox. This includes authentication checks and Git fetch, pull, push, and
remote inspection. Sandboxed credential or network failures are not
authoritative; repeat the command outside the sandbox before diagnosing an
authentication or connectivity problem.

## Graphical session control

Never log out, terminate, restart, or otherwise stop the user's graphical
session on the user's behalf. In particular, do not terminate a login session,
stop its user-session targets, kill the compositor to force an exit, reboot, or
power off the machine. When testing requires a fresh Denial session, tell the
user that a restart is required and wait for the user to log off and return to
SDDM themselves. Continue only after the user confirms that they have logged
back in. Perform session-control actions only when the user explicitly asks
for that exact action.

This restriction does not apply to the dedicated unattended test host
`192.168.1.18` (`.18`). For Denial validation, agents may autonomously stop or
restart its compositor, greetd/login session, launch test applications, and
reboot it when required. Treat `.18` as a disposable lab host, not as the
user's active graphical session.

## Why Denial

**Denial** is an English word. The name contains **Denia**, followed by one
last letter.

It is a quiet reference to Denia from *Wuthering Waves*. Her story never gives
a simple answer to what she originally was, and that uncertainty is important.
What is clear is that others treated her as an asset: something selected,
shaped, and assigned a purpose that was not her own. She was meant to remain a
vessel. Instead, by observing people and learning to live among them, she grew
a heart and gained the ability to choose what she would become.

# PC development build

Denial builds two versioned parts: the Rust compositor in `compositor/` and
the embedded Flutter shell bundle in `dart_shell/`. `tools/denial-pc` keeps
downloaded toolchains and native build output outside the checkout by default.

All `tools/denial-pc` commands must run outside the sandbox as required by
`AGENTS.md`.

Bootstrap the pinned official Flutter SDK and Rust dependencies:

```sh
tools/denial-pc bootstrap
```

Then inspect prerequisites, build and test:

```sh
tools/denial-pc doctor
tools/denial-pc build
tools/denial-pc test
```

The compositor binary is written to
`$XDG_CACHE_HOME/denial/pc-build/rust/release/deniald` by default. The Flutter
bundle is written to `dart_shell/build/linux/x64/release/bundle`.
`tools/denial-pc` builds its AOT assets directly with the locked Denial Flutter
fork and packages the locally rebuilt raw embedder library; normal builds do
not use a third-party platform runner or a C++ Linux runner.

The source lock in `prebuilt/flutter-engine/SOURCE_LOCK.json` pins Denial's
Flutter and Skia forks at exact commits. Their upstream compatibility base is
Flutter `3.44.7`
(`84fc5cbb223bc12f83d65b647ff8a56caf779ffd`), coupled to Dart `3.12.2` and
engine artifact `69c8c61792f04cc809dfef0c910414fb9afc06cd`. All Denial engine,
framework, and Flutter-tool changes live as normal commits in those forks;
this repository must not carry a downstream patch series. Cargo resolves the
exact crate and Smithay revisions in `compositor/Cargo.lock`.

The only editable local engine source roots are:

- Flutter: `/mnt/exty/denial-flutter-fork-3.44.7`;
- Skia: `/mnt/exty/denial-skia-fork-3.44.7`.

The Flutter tree's `engine/src/flutter/third_party/skia` resolves to that Skia
root. Make source changes, run source-formatting work, and create commits only
in these canonical roots. `DENIAL_FLUTTER_SOURCE_ROOT` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skjsbsnq/deniald-win](https://github.com/skjsbsnq/deniald-win) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

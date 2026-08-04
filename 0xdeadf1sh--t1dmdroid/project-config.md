---
trigger: always_on
description: The suite's shared rules and this project's accumulated working knowledge live
---

# T1DMDROID — local rules

## Orient before you edit

The suite's shared rules and this project's accumulated working knowledge live
one directory up, in `../T1DMCOMMON`. Read them first; nothing they hold is
restated here, and this file is only what is local and operational.

- `../T1DMCOMMON/CLAUDE.md` — the suite map, the never-a-second-copy rule, the
  branch policy this file implements, and what may never be published.
- `../T1DMCOMMON/SPEC/invariants.md` — the five-minute grid, `tz_offset`, units
  and sign conventions, **the two risk spaces**, curve semantics, the forecast
  layout, authority and ordering. Normative: where this repository's code
  disagrees with it, one of the two is a defect and neither may be assumed right.
- `../T1DMCOMMON/PROJECTS/T1DMDROID.md` — the target device, the HyperOS traps,
  the build traps, the debugging discipline, the watch link.

`docs/` is the interface documentation — `CGM.md`, `INFERENCE.md`,
`WATCH_BLE.md`, `T1DMSERVER_API.md`. Read the one your change touches, and keep
it true afterwards.

## The skills are gates, not suggestions

Read the description of each before starting, not after being blocked.

- **`publish-audit`** — before `git push`, before adding a remote, before
  creating the repository. Non-negotiable: this repository has been deleted twice
  after private content reached GitHub, and a push cannot be retracted.
- **`terse-ui-text`** — before writing or editing any string a user will see.
- **`android-device-testing`** — the build → deploy → observe loop, the `android`
  CLI, and the `dumpsys`/database patterns that verify behaviour rather than
  pixels.

## Two branches, one body of work

`main` is public and reads the sensor by passive advertisement only. `private` is
local-only and adds everything that is not merely listening: its `:cgm` sources
and their Rust counterpart, the debug bring-up surfaces they need, and the
unredacted sections of `docs/CGM.md`. Only `main` may ever be pushed;
`.git/hooks/pre-push` enforces that as an allowlist, and `publish-audit` greps for
the private symbols by name — so do not name them in a file that lives on `main`,
this one included.

**Everything outside that seam belongs on both branches** — see *T1DMDROID has
two branches, and most work belongs on both* in `../T1DMCOMMON/CLAUDE.md`.
Inference, calculators, alarms, storage, sync, the watch, the UI, the build: a
change to any of it applies to `main` as much as to `private`. Land it as its own
commit on each branch. Never merge or cherry-pick across the seam — that is
precisely how private content reached `main` the first time.

Some differences are **not** drift and must not be reconciled. Diff a file before
mirroring it if it is one of these:

- `.github/workflows/*` — the workflows fire on `push` only on `private`. On
  `main` they are `pull_request`-only, and `publish-audit` fails the push if that
  changes: a `push` trigger there made GitHub build an APK on every push.
- `LICENSE` — carried on `main`, the branch that is published.
- The `:cgm` module's architecture, and every file that plugs into it.

## Build both branches, every time

Both branches get built, every time, and the same invocation serves both:

```sh
PATH="$HOME/.cargo/bin:$PATH" env -u JAVA_HOME ./gradlew :app:assemblePersonalRelease
```

The prefixes are load-bearing. Without `PATH=`, `cargo-ndk` is off the path and
Gradle silently repackages a **stale** `.so` against fresh bindings; without
`env -u JAVA_HOME`, AGP picks up the too-new system JDK. Neither failure is loud.

What differs is the purpose, and therefore what you do with the result.

**`main` — proof that it still compiles.** The public branch has no connected CGM
path, so a change written against `private` can reference a class that does not
exist there, and nothing will notice until the day it is pushed. Build it, read
the result, discard the APK. Add `:app:assemblePublicRelease` when the change
touches flavour-gated code: the `public` flavour compiles a different
`Disclaimer.kt` and stubs the fail-open override out entirely
(`DeathFlavor.SUPPORTED = false`), so it is the only build exercising those paths.

**`private` — the artifact that goes on the phone.** Fully optimized: R8-minified
and resource-shrunk, the variant the user actually runs. This one gets installed.

### Bump the version when it matters

`versionCode` and `versionName` in `app/build.gradle.kts`. Bump both whenever the
build you are about to install differs from the one on the phone in anything the
user would notice — a feature, a fix, a behaviour change — so that the About
screen names what is actually running. A pure refactor that
changes nothing observable does not need one. The bump is its own commit, on both
branches: `Bump to 0.21.4 (versionCode 56).`

### Install it

Check `adb devices` first. If nothing is attached, build anyway, say the install
was skipped, and do not treat an absent phone as a failed task.

If it is attached, locate the APK with `android describe --project_dir .` rather
than guessing the path, then deploy and launch it as `android-device-testing`
describes. Worth knowing before the first install of a session: without a
`keystore.properties` the release build is signed with the **debug** key, so it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xdeadf1sh/T1DMDROID](https://github.com/0xdeadf1sh/T1DMDROID) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->

---
trigger: always_on
description: > `SIGSTOP` is the one signal a process cannot catch, block, or ignore.
---

# sigstop

> `SIGSTOP` is the one signal a process cannot catch, block, or ignore.
> `SIGCONT` resumes it exactly where it left off: registers, memory, file descriptors,
> all intact.
>
> That is what a break is. It is not a restart.

An open-source macOS menu bar app that infers what a developer is *doing*, not just how long
they have been sitting, and interrupts at a defensible moment with a context-aware joke.

## 0. The name is the spec

The product's hardest objection is not "I don't have time." It is **"if I stop now I lose the
stack I've been holding for forty minutes."** The name answers it: a stopped process keeps
everything and continues at the exact instruction. Copy, UI labels and docs all inherit this,
so the vocabulary below is **normative, not decorative.**

| Concept | Signal | Why |
|---|---|---|
| Escalation 1 | `SIGTSTP` | Catchable. You are allowed to ignore it. |
| Escalation 2 | `SIGINT` | Catchable, but ignoring it is rude. |
| Escalation 3 | `SIGTERM` | Catchable. This is your warning. |
| Escalation 4 | `SIGSTOP` | Cannot be caught, blocked, or ignored by anyone, ever. |
| Snooze | `SIGALRM` | Wake me later. |
| Resume from break | `SIGCONT` | The resume button is never labelled "Dismiss". |
| Daily summary | `uptime` | How long you have been going, and how hard. `jobs` was here first and was wrong: it lists suspended work, and this panel shows active time, the longest unbroken stretch, the breaks kept and the app that took the day. `uptime` is a multi-stat readout of exactly that shape, and no developer has to look it up. |
| Reload settings | `SIGHUP` | Its honest daemon meaning: re-read the config. |

**Two hard rules, from the naming review:**

1. **`SIGKILL` never appears at level 4, or anywhere.** SIGKILL is unrecoverable and destroys
   exactly the thing the name promises to preserve. The ladder tops out at `SIGSTOP`, which is
   already uncatchable *and* destroys nothing. Using SIGKILL for a laugh contradicts the product.
2. **`SIGHUP` is never an escalation rung.** Its default disposition is *terminate*, so an L1
   labelled SIGHUP would quietly mean "die." It is reserved for reloading settings.

**The tone risk this name carries.** The metaphor casts the app as the kernel and the developer
as an uncooperative process. Played straight, that is an authority scolding the user, which is
the relationship people uninstall. The app must stay self-aware that *"cannot be ignored" is a
bluff the user is in on*: a menu bar app cannot suspend anyone. The app is on your side; it is
the thing that guarantees you come back intact. Write it as a deadpan accomplice, never a warden.

This file is the operating manual for anyone (human or agent) working in this repo.
Read it before editing. The rules in **Invariants** are not stylistic preferences.

---

## 1. Repo layout

```
sigstop/
├── app/          macOS app: Swift 6, SwiftUI, SwiftPM (no .xcodeproj)
├── docs/         Architecture. Written before the code and kept in sync.
└── CLAUDE.md
```

**This repository is the app.** The landing site was split out and lives in
[`sigstop-web`](https://github.com/Mohamed-Elshesheny/sigstop-web). They share a name and nothing
else: different language, toolchain, audience and release cadence. Nothing here builds, serves or
tests the site, and a change to the page does not belong in a pull request here.

One thing does cross the line, and §5 says what to do about it: the ten badge names are printed by
both.

Design docs are normative, not aspirational. If code and `docs/` disagree, one of them is a bug.
Decide which, fix it, and say so in the PR.

| Doc | Owns |
|---|---|
| `docs/ACTIVITY-DETECTION.md` | Signal tiers, providers, the confidence model |
| `docs/BREAK-DECISION.md` | Session clock, engine state machine, interruption policy |
| `docs/MESSAGE-ENGINE.md` | Template selection, tone, escalation, corpus format |
| `docs/PRIVACY.md` | Data inventory and the enforceable no-collection properties |
| `docs/RELEASING.md` | Cutting a release: EdDSA signing, the appcast, and publishing |

---

## 2. Build and run

```sh
cd app
make build          # swift build -c release
make bundle         # assemble + sign sigstop.app
make run            # bundle, then launch
make test           # badge check, then swift test: Core only, no GUI session required
make doctor         # print exactly what the app can observe right now
make verify         # assert the §4.3 claims against the BUILT bundle, not the source
make badges         # rewrite Exports/badges.json from the Swift catalogue (§5)
```

There is **no Xcode project and no Xcode requirement**. Command Line Tools are enough.
`swift build` compiles SwiftUI and AppKit fine; `make bundle` assembles the `.app` by hand.
Do not add a `.xcodeproj`. It breaks CI and produces merge conflicts for no gain.

---

## 3. Architecture rules

### 3.1 Dependency direction is one-way

```
App  ──▶  Sensors  ──▶  Core
```

- **`Core`** is pure domain. **Must not import AppKit, Cocoa, or any macOS UI framework.**
  Session clock, decision engine, message engine live here.
- **`Sensors`** is the *only* layer that touches macOS APIs. Everything it exposes is a protocol.
- **`App`** is the SwiftUI menu bar, the break overlay and settings.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mohamed-Elshesheny/sigstop](https://github.com/Mohamed-Elshesheny/sigstop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->

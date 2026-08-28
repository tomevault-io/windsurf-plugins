---
trigger: always_on
description: validates. One wrong offset and 27 silently drops the swipe — no error, just no switch.
---

# Working on noswoosh

One Swift file (`noswoosh.swift`), a bundle script, and a release workflow. Read the
source comments first — they explain the technique. This file covers only what the
code can't tell you.

## Shape

Two input sources — the Ctrl+arrow hotkey and an event tap that intercepts real
3-finger swipes — both call one `switchSpace(right:)` core. The core has two posting
paths chosen by `needsAugmentation` (runtime `kern.osproductversion >= 27`): the
lightweight pre-27 path (verified on macOS 26), and the macOS 27+ path that attaches a
serialized IOHID payload. Keep new work behind that gate so a change to one OS can't
regress the other. Verified on macOS 26 and 27 only — don't claim older releases the
pre-27 path *should* handle but nobody has tested.

## Build and release

```sh
swiftc noswoosh.swift -O -o noswoosh -F /System/Library/PrivateFrameworks -framework SkyLight
./scripts/make-app-bundle.sh --out build     # assembles noswoosh.app
```

Releasing is a tag push: bump `noswooshVersion` in `noswoosh.swift`, then
`git tag vX.Y.Z && git push origin vX.Y.Z`. CI builds, signs, notarizes, staples,
publishes, and bumps the cask in `mmathys/homebrew-tap`. The workflow header lists the
secrets; each group degrades to a skip when absent. Only edit the tap by hand if the
bump step reported a skip or a warning.

The tap release is fully automatic: the tag push triggers CI, which opens/merges the
cask bump in `mmathys/homebrew-tap` — never clone or push that repo by hand.

## Traps

**Don't tidy the private-API constants.** The numeric `CGEventField`s and the `1e-4`
gesture progress are load-bearing and hard-won. `FLT_TRUE_MIN` — what the reference
implementations use — is flushed to zero on Apple Silicon and breaks direction. Both
paths use `1e-4` as of 1.7.1: the 27 path shipped `±1.0` (full travel) through 1.7.0,
which switched correctly and so passed every functional test, but *visibly slid* —
instant switching is the whole point, and no assertion in this repo catches a switch
that works but animates. The ±9999 fling on `.ended` is what commits the 27 swipe, so
only the sign of progress matters; don't use `0`, which `fixed1616` serializes as 0 in
the IOHID payload. Verify by eye on a real 27, not just by checking that it switched.

**The macOS 27 IOHID payload is byte-exact.** `generateIOHIDPayload` writes packed
little-endian structs (28/40/28-byte records) whose sizes and field offsets the Dock
validates. One wrong offset and 27 silently drops the swipe — no error, just no switch.
Don't "clean up" the manual byte writes into Swift structs; Swift doesn't guarantee C
packing. If you touch it, re-verify on a real 27 (see VM testing below), not just a
compile.

**The passthrough counter couples the core and the tap.** Every synthetic event the
core posts re-enters our own event tap. The core bumps `passthrough` by exactly the
number of events it posts (1 per bare event, 2 per augmented pair); the tap decrements
and passes those through instead of re-intercepting. If you change how many events a
post emits, update the bump in lockstep or the tap will eat its own output or act on it
twice.

**macOS 27 reverses swipe direction.** `isRightSwipe` and `makeAugmentedDockEvent` flip
the progress/velocity sign versus the pre-27 path. If direction is backwards on one OS
but right on the other, this is why — check the `needsAugmentation` branch, not the
field constants.

**The daemon must be `.accessory`, not `.prohibited`.** The yank guard works by
taking activation the moment we land on a space with no windows. A `.prohibited` app
cannot become active at all, so "tidying" the policy back silently reintroduces the
empty-desktop yank with no error and no log line. Neither policy shows a Dock icon or
a Cmd-Tab entry, so the change is invisible until you test on an empty desktop. Note the guard only
runs on macOS < 27 (`yankGuardNeeded`), so testing this on a 27 box proves nothing —
use `NOSWOOSH_FORCE_YANK_GUARD=1` there, or test on 26.

**The yank guard has to fire on landing, not before.** Taking activation ahead of the
switch does nothing — the switch re-activates macOS's own pick when it commits. And
parking a real window on the destination space doesn't help either; emptiness is what
triggers the hunt, but the yank comes from the *other* app's window ordering. Both
were measured; see the README section.

**Never `cp` over the running binary.** `cp` rewrites the destination inode in place.
Do that to a running, signed binary and the kernel keeps page hashes that no longer
match it, then SIGKILLs every subsequent exec of that inode — new processes included.
The symptom is maddening: `codesign -v --strict` passes, the hash matches the build
byte for byte, the same bytes run fine from another path, and launchd just reports
`-9`. `scripts/install.sh` boots the agent out and `rm -f`s the target first; keep it
that way, and reach for `rm`-then-copy (or a temp file plus `mv`) anywhere else.

**Accessibility trust is cached for a process's lifetime.** That is the entire reason
the daemon polls `AXIsProcessTrusted()` and exits once granted, letting launchd's
`KeepAlive` restart it. It looks like a redundant loop; it isn't. Removing it brings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmathys/noswoosh](https://github.com/mmathys/noswoosh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

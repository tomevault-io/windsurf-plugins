---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

simslim runs many more iOS simulators on one Mac by disabling the background
daemons a simulator doesn't need, cutting each simulator's memory ~4x. It is a Go
CLI plus a SwiftUI macOS app that wraps it. Everything is driven through
`xcrun simctl`; the tool only ever touches the simulators you point it at, never
the host Mac. macOS-only.

## Commands

```sh
go build ./cmd/simslim          # build the CLI
go test ./...                   # run all tests (Makefile: make test)
go test -run TestName ./...     # run a single test
make check                      # full CI gate — must pass before a PR
make format                     # gofmt + swift-format (run before committing)
make app                        # build build/SimSlim.app (Go + swiftc, macOS only)
open build/SimSlim.app
```

`make check` runs `go test`, `go vet`, `swift-format lint --strict --recursive gui`,
`zsh -n scripts/build-app.sh`, and `plutil -lint gui/Info.plist`. CI (`.github/workflows/ci.yml`)
runs exactly this on macos-26 / Xcode 26.6, then builds and verifies the app bundle.

Tests are pure unit tests (parsing, delta logic, allowlist invariants) — they do
**not** boot real simulators, so they run anywhere. Only the app build and manual
runs need Xcode + an iOS runtime.

## Architecture

Two packages. The repo root is `package simslim`, an importable library holding
every piece of slimming logic and **no external dependencies**. `cmd/simslim/`
is `package main`, the CLI: `main.go` dispatches `os.Args[1]` to a `cmd*`
function per subcommand and enforces macOS-only up front.

Anything that talks to a terminal — printing, `--json` encoding via `writeJSON`,
the interactive wizard in `wizard.go`, `fatal()`, `usage()` — lives in
`cmd/simslim/`. The library never writes to stdout; it returns values and
reports progress through the `Reporter` callback the caller supplies.

Exported identifiers in the root package are the public API, so renaming one is
a breaking change for importers as well as for the CLI.

**The slimming model (the core idea).** `profiles.go` defines `Categories`, an
allowlist of launchd daemon labels grouped by user-facing feature (siri, search,
icloud, …). Categories may overlap — a label lives in every category whose
feature needs it (e.g. the AMS payment-sheet daemons are in both `store` and
`icloud`), and `Profile.Desired()` keeps a label enabled when **any** excepted
category lists it. `SlimmableSet()` is the deduplicated union of every label in
`Categories`.
`managedSet()` adds each category's `AlwaysEnabled` compatibility services,
which simslim may only repair back to enabled; these are **the only labels the
tool may ever disable or enable.** Anything outside those sets is never touched.
`service_descriptions.go` supplies the short per-daemon explanations shown by
the GUI; its coverage and length are enforced in `profiles_test.go`.
`profile_file.go` loads a committed JSON profile (`simslim on --profile <path>`)
whose `except`/`keep` arrays mirror the flags of the same name, validates it
against the allowlist, and resolves it to a `Profile`. The dependency-free
`profile` command's interactive wizard lives in `cmd/simslim/wizard.go`.
`features.go` defines `Features`, a finer-grained catalog than `Categories`:
each feature (push, storekit, universal-links, …) names just the daemons one
testable capability needs. `doctor` reads a booted simulator's disabled labels
and reports any required feature whose daemons are down, exiting non-zero — a CI
preflight. `features_test.go` asserts every feature label is slimmable.
`slim.go`'s `ensure()` rejects a non-empty slim profile on runtimes older than
iOS 18.5 before booting or mutating the device, then reads the currently disabled
labels, computes a `delta` against the desired set, and applies the changes with
`launchctl disable/enable` run inside the simulator via `simctl spawn`, a pool of
`spawnWorkers` (8) at a time. It reboots
and reads the state back before reporting persistence. `on` disables the profile;
`off` remains available on every runtime and re-enables the whole managed set.
`EnableSlimNoReboot` (`on --no-reboot`) skips the reboot: it runs `launchctl
disable` + `launchctl bootout` per label so the daemon stops in the current boot
session, which is the only slimming possible on runtimes older than iOS 18.5.

**The offline fast path.** On a runtime with persistent overrides `ensureOffline`
takes over: `disabled_store.go` writes the overrides directly and boots the
device once, already slim, skipping every `launchctl` spawn *and* the reboot that
would apply them (measured 2m14s → 33s for 170 labels). A **booted** device gets
there too — an override only takes effect at the next boot, so `ensure` reads its
live state, returns early when the profile already matches, and otherwise spends
the shutdown the device already owed before handing off. There is no shell inside
any iOS runtime (`RuntimeRoot/bin` holds only `df` and `launchctl`), so batching
transitions through a spawned `/bin/sh` is not an option; the pool is. `launchd_sim` is a *host* process, so that store is not in the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MobAI-App/simslim](https://github.com/MobAI-App/simslim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->

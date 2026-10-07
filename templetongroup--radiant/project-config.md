---
trigger: always_on
description: The repo's root `AGENTS.md` still applies. Radiant loads this file first and the root one only
---

# Radiant for iPhone — read this before touching `apps/ios`

The repo's root `AGENTS.md` still applies. Radiant loads this file first and the root one only
partly, so the root's shipping rule, in short: **every change is committed and
pushed, gets a phone Read me entry (`Native/ReadMeView.swift`), and a
Linear issue in TG / Radiant — then `node scripts/ship-check.mjs`.** A phone
change also needs a new build on TestFlight (`scripts/ios-testflight.sh`) and
on every device (`scripts/ios-install-all.sh`).

## App Store status

⚠️ **STILL READ APP STORE CONNECT BEFORE YOU TRUST THIS.** `node
scripts/asc.mjs get 6804891721` prints the live state in one line; this
heading has gone stale three times. 1.0 (build 7) was approved 2026-09-16; 1.1
(build 21) was submitted 2026-09-17 from the command line (`asc.mjs submit`)
and approved the next morning with no questions.

**1.1 carried:** Hugging Face search with a run/won't-run verdict (unfiltered
— TG-454, do not reinstate a word filter), Archive on chats, the keyboard fix,
the unsent-message fix (TG-467), the byline link to templetontech.com, and an
age rating of 17+ — answered honestly for an open model list.

**1.2 is being prepared** (PREPARE_FOR_SUBMISSION on 2026-09-27): the native
SwiftUI rebuild (TG-567), subscription sign-in (TG-570), Jev Router (TG-571),
builds 29–38 on TestFlight. It waits on Tony testing on his phone. Read
`CURRENT_PROJECT_VERSION` in the project file — every upload must be higher.

**Each version is its own review:** a version number in App Store Connect
(`asc.mjs new-version` made 1.2), its own what's new, its own review. Screenshots are
still 1.0's — replace them with the next submission (`asc.mjs shots` counts
them). `scripts/asc.mjs` can do everything short of signing in.

**TestFlight from the command line: `scripts/ios-testflight.sh`** (bump
`CURRENT_PROJECT_VERSION` first). It signs LOCALLY — an Apple Distribution
certificate created with the API key on 2026-09-25 (expires 2027-09-25), its key
in `~/Library/Keychains/radiant-signing.keychain-db`, profile "Radiant App Store
(command line)" — and uploads with `altool`, so it works with nobody signed in
to Xcode. Xcode's own upload needs an Apple ID in Settings → Accounts; the API
key is refused cloud signing. The Internal TestFlight group gets every build.

**Anything to do with the submission: use the `app-store-review` skill**
(`.claude/skills/app-store-review/`, also installed at `~/.claude/skills`;
published at https://github.com/templetongroup/app-store-review — the repo is
the copy people install, so a change here goes there too). It is the whole
adventure — both rejections, the TestFlight false alarm, the privacy Publish
button, the two-button resubmit — turned into a protocol.

**The catalogue is published, not only compiled in.** `apps/ios/catalog.json` is
fetched at launch and applied over the built-in Swift array, so a broken row can
be corrected in minutes instead of a review cycle. It is GENERATED from that
array (`npm run catalog:export`), so the two cannot drift, and every failure
falls back to what shipped.

⚠️ **That also means a bad publish reaches every phone at once.** `npm run
catalog:publish` runs the export, then `scripts/catalog-check.py`, which probes
every repo and refuses on undeclared quantization, a size more than 10% off the
real blob total, or a 404. Do not copy catalog.json to the website by hand.
**Before any future submission, run `npm run catalog:check`.** It fails any repo
under ~1.2 bytes per parameter that declares no quantization — the Gemma 4
defect, which shipped because the old check only asked whether MLX implemented
the architecture. It also builds every row's real config.json with the phone's
engine (the pinned one, and for main-list rows the oldest one still on phones —
App Store 1.1's), which is what would have stopped Nemotron 3 Nano 4B shipping.

## The app, the build and the devices

`apps/ios` is a native SwiftUI app (`apps/ios/ios/App/App/Native/`). The
Capacitor project remains only as the Xcode shell and for the `LocalModels`
engine class; no web view is shown and `src/mobile` is gone (build 42,
2026-09-28 — Tony: "get rid of the old design").

**Every iOS build goes to every device.** Standing instruction from Tony
(2026-09-10): *"when you create new builds to the ios version, i want you to
update it on all devices."* A dev install only changes when someone pushes a
new one to that device, so a build that lands on one phone leaves the others
on last week's code with no way to tell. One command does the whole job —
web bundle, sync, build once, install on every paired device that answers:

```bash
scripts/ios-install-all.sh
```

It lists the devices that did not answer (off, asleep, not on this network)
at the end; run it again when they are. Devices today: iPhone 17 Pro Max,
iPad Pro 11, iPad mini (A17 Pro). All are on the paid team's profile, which
lasts a year — not the seven days a free Apple ID gets.

⚠️ **The store is a file, and its first launch imports the web's.** `KV`
(NativeKit.swift) holds the same `radiant.phone.*` / `rx.*` keys and JSON
shapes the old web design kept in localStorage, backed by
`Application Support/radiant-store.json` (`Launch.swift`, `DiskStore`). The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [templetongroup/radiant](https://github.com/templetongroup/radiant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

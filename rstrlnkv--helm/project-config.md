---
trigger: always_on
description: Orders, each with the reason it exists. `ARCHITECTURE.md` is the twin that describes how Helm is built; nothing here repeats it. A rule that arrives without a reason is litter — delete it.
---

# Helm — how to work here

Orders, each with the reason it exists. `ARCHITECTURE.md` is the twin that describes how Helm is built; nothing here repeats it. A rule that arrives without a reason is litter — delete it.

## Commands

- Run `bash Scripts/test.sh` for the whole suite and `bash Scripts/test.sh --filter '<Name>|<Other>'` for one family — a filtered run is seconds against minutes, which is what makes running a guard before the suite cheap enough to actually do; the script passes its arguments straight through to `swift test`.
- Read a run through `bash Scripts/test.sh`, which logs the whole output, reads the exit status off its own line and prints its own verdict last — the closing "Test Suite 'All tests' passed" speaks for the last bundle and the very last line, "✔ Test run with 0 tests in 0 suites passed", for Swift Testing's empty run — neither for the suite, a summary line that also carries a skip count reads as zero failures under a shorter pattern, and a `--filter` whose alternatives match nothing, or match only in part, still exits 0; the runner counts each alternative against the cases that actually ran and fails, naming the one that ran none. Where a bare `swift test` log has to be read, take the status on its own line — `swift test > log 2>&1; echo "EXIT=$?"` — and count `command grep -cE 'with ([0-9]+ tests? skipped and )?[1-9][0-9]* failures?' log` and `command grep -c 'warning: No matching test cases were run' log`; a filter that matches in part is invisible to both.
- Build one target while working with `swift build --target <Name>`, and a release only as `swift build -c release --product HelmApp` — a bare `swift build -c release` also builds the test-support target, which imports XCTest out of the Xcode-only framework path and lands beside the app; `Tests/HelmAppTests/ReleaseBuildsTheProductTests.swift` holds both halves.
- Package with `bash Scripts/package-app.sh` and install only from the signed copy it stages under `$TMPDIR` — that is the bundle the script signed and verified, while `.build/` holds an unsigned one with no usable identity at all.
- Use `bash Scripts/package-dev.sh` for a build to click through by hand — it installs beside the real app with its own bundle id and its own settings domain, so a manual check does not cost the installed app its grants.
- Read the shipping version with `/usr/libexec/PlistBuddy -c 'Print :CFBundleShortVersionString' Resources/HelmApp/Info.plist` — that key is the number the updater compares, and every other place the number appears is derived from it.
- Regenerate `Resources/DesignSystem/design-tokens.json` with `HELM_WRITE_DESIGN_TOKENS=1 bash Scripts/test.sh --filter PublishedTokensAreTheTrees` after a token moves, and read the run as a failure that wrote a file rather than as a failure — that mode never passes, because a mode that rewrites its own expectation and then reports success is a check that cannot fail; the record is what the design system published outside this repository is built from, so a change here is only half a change until that is republished.
- Run `plutil -lint Sources/HelmUI/Resources/*.lproj/Localizable.strings` after any hand edit — a malformed file is silent, and every string in it falls back to English with no error anywhere.
- Run `bash Scripts/test.sh --filter 'PunctuationIsTerminologyTests|StringsCoverageTests|NoOrphanTranslationsTests'` after touching a `.strings` file, before the suite — those three are the guards a string change trips, they finish in seconds, and the runner fails, naming it, if one of the three stops matching a test.
- Keep `command grep -rn HELM_DEBUG Sources/` empty before every commit — the visual harness is env-gated and belongs in the working tree only while it is being used.
- Answer "who uses this" from an index and never from `grep` — this tree writes backticked names inside doc comments deliberately and at volume, so a grep counts prose as a caller; run `swift build --build-tests`, then `periphery scan --skip-build --index-store-path "$PWD/.build/out" --format csv`. Do not read that report as a to-do list either — its "unused" covers dead code, a marker a test checks by type rather than by call, and a fake's capability no test has needed yet, while its assign-only findings are usually a property held to keep an object alive, a token whose clearing is the cancellation, or a field a synthesized conformance reads.
- Read `~/Library/Logs/Helm/helm.log` rather than assuming — its `[permissions]` line is where the installed build says which grants it actually holds.
- Measure before believing a cleanup — `du -sh "$TMPDIR"` first, then check what under there is not `helm-*`, because the same folder holds other programs' files; pass `--scratch-path` only when another suite run may be up and reuse one path for the session, since a fresh directory per invocation is over a gigabyte that nothing sweeps while the machine stays up.

## Before you touch the code


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rstrlnkv/Helm](https://github.com/rstrlnkv/Helm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

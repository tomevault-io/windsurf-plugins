---
trigger: always_on
description: This is a Swift Package macOS app for RepoPrompt CE.
---

# Agent Notes

This is a Swift Package macOS app for RepoPrompt CE.

Prefer the coordinated developer daemon (`make dev-*`, see "Developer daemon / coordinated validation" below) for builds, runs, and tests. It runs every job through a lane-serialized queue so concurrent agents do not build, launch, or test over each other, and it returns a ticket for each job so long builds can be detached and checked on later instead of blocking. The plain `make` / `swift` / `./Scripts` commands shown below are the uncoordinated fallback for when the daemon is unavailable.

## Contribution preflight

Before every commit or push, read and run the repository-local `$rpce-contribution-check` skill:

```bash
.agents/skills/rpce-contribution-check/scripts/preflight.sh commit
.agents/skills/rpce-contribution-check/scripts/preflight.sh push
```

Stage only the intended changes, then use `commit` mode before creating a commit; rerun it after any staging change, including partial-staging updates. Use `push` mode after committing but before pushing the intended current branch. These default modes are mandatory safety gates: they enforce redacted staged-index and outgoing-range secret scanning, repository guardrails, and clean push boundaries. Default `push` does not run heavyweight lint/test/build lanes; use the explicit `pr-ready` lane when you need the computed-outgoing-range path-selected local PR-ready pass:

```bash
.agents/skills/rpce-contribution-check/scripts/preflight.sh pr-ready
```

Focused validation and release validation remain explicit; use the validation matrix plus commands such as `make dev-release-preflight` / `make dev-release-artifact` when the changed boundary requires them. Obtain explicit user approval immediately before any force-push, history rewrite, branch deletion, fork deletion, credential rotation, other GitHub-visible destructive mutation, visible app launch/relaunch, or stopping a visible app.

Local `docs/investigations/*.md` reports are intentionally left unignored so RepoPrompt tooling can read them. Do not stage or merge these local investigation artifacts unless intentionally requested.

## Run

```bash
make doctor     # verify Swift/Xcode command line tool setup, SDK, signing diagnostics, SwiftUI probe, and debug CLI status
make dev-run    # coordinated build, package, stop existing RepoPrompt, and launch the debug app
```

`make dev-run` routes through the developer daemon (see "Developer daemon / coordinated validation") and remains the ordinary FIFO coordinated launch path. For a user-directed newest lifecycle action, use `./conductor app relaunch`; the Finder launcher uses that operation when `python3` is available. The uncoordinated equivalents are `make run` or `./Scripts/run.sh`.

Debug packaging may auto-detect an Apple Development signing identity for a valid local app signature, but auto-detected debug signing still uses ephemeral in-memory secure storage to avoid macOS Keychain prompts. Set an explicit `SIGN_IDENTITY="Apple Development: ..."` to opt in to persistent debug Keychain storage; `DEBUG_SECURE_STORAGE_BACKEND=keychain` is also supported for explicit debug storage opt-in when the signed app has a TeamIdentifier. If no stable identity is available, set `ALLOW_ADHOC_SIGNING=1` to build an ad-hoc debug app; ad-hoc debug builds use ephemeral in-memory secure storage, so API keys and secure permission changes do not persist across launches. Release packaging requires `SIGN_IDENTITY` and continues to use real Keychain storage.

## Debug

Package without launching:

```bash
make dev-build                  # coordinated debug package (preferred)
# uncoordinated equivalents:
make build
./Scripts/package_app.sh debug
```

Use verbose shell tracing when packaging fails or hangs:

```bash
VERBOSE=1 ./Scripts/package_app.sh debug 2>&1 | tee /tmp/repoprompt-build.log
```

The debug app bundle is created through:

```text
.build/debug/RepoPrompt.app
```

SwiftPM’s architecture-specific build output is usually under:

```text
.build/arm64-apple-macosx/debug/
```

## Generated Xcode workspace

The repository-owned workspace is a disposable developer convenience under
`.build/xcode`; `Package.swift`, conductor, and `Scripts/package_app.sh` remain
authoritative.

```bash
make xcode                  # generate and open
make xcode-generate         # generate without opening
make xcode-generator-test   # deterministic generator contract tests (default CI)
make xcode-validate         # explicit full validation with xcodebuild -list
make xcode-clean            # remove generated workspace metadata
```

Default CI runs `make xcode-generator-test`; full `make xcode-validate` is explicit and runs through local `pr-ready` for Xcode workspace boundary changes or the dedicated `Xcode Workspace Validation` workflow.

Xcode 26.3 exposes the native `RepoPrompt` and `repoprompt-mcp` product schemes.
Use `RepoPrompt CE App` and `RepoPrompt CE MCP` for conductor-coordinated debug
products. `RepoPrompt CE Tests` delegates to conductor because `RepoPromptMCP`
is executable-only and cannot back a native Xcode unit-test dependency. Do not
edit or commit `.build/xcode`, use these schemes for release/archive work, or
assume canceling Xcode cancels a queued conductor job; inspect

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [repoprompt/repoprompt-ce](https://github.com/repoprompt/repoprompt-ce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->

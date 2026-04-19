---
trigger: always_on
description: **The free AI already on your Mac.** This is our claim. Every surface (README, landing page, repo description) must reinforce it.
---

# apfel - Project Instructions

**The free AI already on your Mac.** This is our claim. Every surface (README, landing page, repo description) must reinforce it.

## The Golden Goal

apfel has ONE purpose with THREE delivery modes:

> **Expose Apple's on-device FoundationModels LLM as a usable, powerful UNIX tool
> and an OpenAI API-compatible server, with a working command-line chat.**

### The three modes, in priority order:

1. **UNIX tool** (`apfel "prompt"`, `echo "text" | apfel`, `apfel --stream`)
   - Pipe-friendly, composable, correct exit codes
   - Works with `jq`, `xargs`, shell scripts
   - `--json` output for machine consumption
   - Respects `NO_COLOR`, `--quiet`, stdin detection

2. **OpenAI-compatible HTTP server** (`apfel --serve`)
   - Drop-in replacement for `openai.OpenAI(base_url="http://localhost:11434/v1")`
   - `/v1/chat/completions` (streaming + non-streaming)
   - `/v1/models`, `/health`, tool calling, `response_format`
   - Honest 501s for unsupported features (embeddings, legacy completions)
   - CORS for browser clients

3. **Command-line chat** (`apfel --chat`)
   - Interactive multi-turn with context window protection
   - Typed error display, context rotation when approaching limit
   - System prompt support

The Debug GUI has been extracted to its own repo: [apfel-gui](https://github.com/Arthur-Ficial/apfel-gui)

### Non-negotiable principles:

- **100% on-device.** No cloud, no API keys, no network for inference. Ever.
- **Honest about limitations.** 4096 token context, no embeddings, no vision - say so clearly.
- **Clean code, clean logic.** No hacks. Proper error types. Real token counts.
- **Swift 6 strict concurrency.** No data races.
- **Usable security.** Secure defaults that don't get in the way.
- **TDD always, red-to-green, 100%.** No production code without a failing test first. Write the test, watch it fail for the right reason, write the minimal code to pass, watch it go green. No exceptions, no "I'll add tests after", no "this is too simple to test". Behavior-preserving refactors are covered by existing tests; new behavior gets a new failing test first.

### Documentation style:

- **Links in docs and README:** Always use the URL/path as the anchor text, not generic phrases like "full guide" or "click here". Example: `[docs/background-service.md](docs/background-service.md)` not `[full guide](docs/background-service.md)`.

## Architecture

```
CLI (single/stream/chat) ──┐
                           ├─→ Session.swift → FoundationModels (on-device)
HTTP Server (/v1/*) ───────┘   ContextManager → Transcript API
                                ContextStrategy → 5 trimming strategies
                                SchemaConverter → DynamicGenerationSchema
                                TokenCounter → real tokenCount (SDK 26.4)
```

- `ApfelCore` library: pure Swift, no FoundationModels dependency, unit-testable
- Main target: FoundationModels integration, Hummingbird HTTP server
- Tests: `swift run apfel-tests` (pure Swift runner, no XCTest needed)
- No Xcode required - builds with Command Line Tools only

## Current Status

- Version: `1.0.0` (source of truth: `.version`)
- Tests: 366 unit + 220 integration
- Distribution: homebrew-core (`brew install apfel`), nixpkgs (`nix profile install nixpkgs#apfel-ai`), and the Arthur-Ficial/homebrew-tap
- Stability policy: [STABILITY.md](STABILITY.md)
- Security policy: [SECURITY.md](SECURITY.md)

## Build & Test

```bash
make test                      # BUILD + ALL TESTS (unit + integration) - the one command you need
make install                   # build release + install to /usr/local/bin (NO version bump)
make build                     # build release only (NO version bump)
make version                   # print current version
swift build                    # debug build
swift run apfel-tests          # unit tests only (366 tests)
make preflight                 # full release qualification (unit + integration + policy checks)
```

`make test` builds the release binary, runs all 366 unit tests, starts test servers, runs all 220 integration tests, and cleans up. This is the single command for development.

`make install` auto-unlinks Homebrew apfel so the dev binary takes PATH priority. `make uninstall` restores the Homebrew link.

**Version is in `.version` file** (single source of truth). Local builds (`make build`, `make install`) do NOT change the version. Only the release workflow (`make release`) bumps versions. This ensures patch versions mean "published compatible fix", not "someone ran a build". **Never manually edit `.version`, `BuildInfo.swift`, or the README badge** - these are updated atomically by the release workflow.

Regenerate `docs/EXAMPLES.md` (runs 53 prompts against the installed binary, captures real unedited output):
```bash
bash scripts/generate-examples.sh          # ~2 minutes, overwrites docs/EXAMPLES.md
```

## Key Files

| Area | Files |
|------|-------|
| Entry point | `Sources/main.swift` |
| CLI commands | `Sources/CLI.swift` |
| HTTP server | `Sources/Server.swift`, `Sources/Handlers.swift` |
| Session mgmt | `Sources/Session.swift`, `Sources/ContextManager.swift` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arthur-Ficial/apfel](https://github.com/Arthur-Ficial/apfel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->

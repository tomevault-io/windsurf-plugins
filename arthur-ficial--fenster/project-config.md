---
trigger: always_on
description: **The free AI already in your browser, served as if it were OpenAI.** This is our claim. Every surface (README, landing page, repo description) must reinforce it.
---

# fenster - Project Instructions

**The free AI already in your browser, served as if it were OpenAI.** This is our claim. Every surface (README, landing page, repo description) must reinforce it.

## The Golden Goal

fenster exposes Chrome's on-device Gemini Nano via the Prompt API, behind the same OpenAI-compatible HTTP server that apfel exposes. **Two things are the product. One thing is a byproduct.**

### Core product (this is what fenster IS)

1. **UNIX tool** (`fenster "prompt"`, `echo "text" | fenster`, `fenster --stream`)
   - Pipe-friendly, composable, correct exit codes
   - Works with `jq`, `xargs`, shell scripts
   - `--json` output for machine consumption
   - Respects `NO_COLOR`, `--quiet`, stdin detection

2. **OpenAI API-compatible HTTP server** (`fenster --serve`)
   - Drop-in replacement for `openai.OpenAI(base_url="http://localhost:11434/v1")`
   - `/v1/chat/completions` (streaming + non-streaming)
   - `/v1/models`, `/health`, tool calling, `response_format`
   - Honest 501s for unsupported features (embeddings, legacy completions)
   - CORS for browser clients
   - **Same wire format as apfel.** Verified by running apfel's integration test suite verbatim against fenster (see "The apfel-compat gate" below).

These two modes are what the README.md leads with. Every design decision, test, and release gate is scored against them first.

### Byproducts (useful, but not the pitch)

3. **Interactive mini TUI chat** (`fenster --chat`) — **a byproduct for quick testing, not a main product.**
   - Ships because the pieces are already there (Session, ContextManager, tool calling)
   - Handy for quick testing a prompt or a local MCP server without writing a client
   - Should not dominate README real-estate; a short Quick Start entry is enough

### README.md structure rule

The README.md mirrors this priority — **violating this structure is a bug.**

- Hero + tagline: UNIX tool and OpenAI-compatible server only
- "What it is" table: **two rows** (UNIX tool, OpenAI server). Nothing else.
- Right after the table: a one-command "Try it right away: `fenster --chat`" pointer. Rationale: chat is not the main product, but it is the lowest-friction way for a new user to verify install and see fenster responding.
- Quick Start: UNIX tool first, server second, chat gets a short subsection
- Reference Docs: links to platform matrix and architecture notes

### Non-negotiable principles

- **On-device.** No cloud, no API keys, no network for inference after the initial Gemini Nano download. Ever.
- **Honest about limitations.** Chrome required. ~3B model. No embeddings. Tool calling is faked via `responseConstraint` — say so clearly.
- **Clean code, clean logic.** No hacks. Proper error types. Real token counts (or honest estimates with `_estimated: true`).
- **Modern Go.** Go 1.22+. Stdlib-first. No third-party HTTP router. `log/slog` for logging. `embed.FS` for the extension. `context.Context` everywhere.
- **Usable security.** Secure defaults that don't get in the way. CORS off by default. Bind to `127.0.0.1` by default. No bearer tokens over `http://` external interfaces.
- **TDD always, red-to-green, 100%.** No production code without a failing test first. Write the test, watch it fail for the right reason, write the minimal code to pass, watch it go green. No exceptions, no "I'll add tests after", no "this is too simple to test". Behavior-preserving refactors are covered by existing tests; new behavior gets a new failing test first.

### Documentation style

- **Links in docs and README:** Always use the URL/path as the anchor text, not generic phrases like "full guide" or "click here". Example: `[docs/native-messaging.md](docs/native-messaging.md)` not `[full guide](docs/native-messaging.md)`.

## The apfel-compat gate

fenster's load-bearing acceptance criterion: **the entire apfel integration test suite, vendored verbatim, must pass against fenster's binary.**

- The apfel pytest suite lives at `Tests/integration/`. It was copied from `Arthur-Ficial/apfel` and patched only at one point: `conftest.py` spawns `bin/fenster` instead of `.build/release/apfel`.
- The suite is transport-agnostic — it talks HTTP/SSE/JSON to `localhost:11434` and `localhost:11435`. It does not care which language wrote the server. If fenster's wire format diverges, a test breaks.
- Apfel-specific tests that don't apply (`test_apfelcore_*`, `test_brew_service`, `test_nixpkgs_bump`) are excluded by name in `Tests/integration/conftest.py::collect_ignore`.
- **Skipping a test is a critical error.** `pytest.skip()` calls in the suite are forbidden in green state. The release gate counts skipped tests as failures.
- When the upstream apfel suite changes, we re-vendor (`scripts/port-apfel-tests.sh`) and accept any new red tests as work.

## Architecture

```
CLI (single/stream/chat) ──┐
                           ├─→ HTTP server (/v1/*)  ──┐
                                                       ├─→ NM stdio (4-byte LE prefix + JSON)
                                                       ├─→ Chrome extension service worker
                                                       ├─→ LanguageModel.create() / promptStreaming()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arthur-Ficial/fenster](https://github.com/Arthur-Ficial/fenster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->

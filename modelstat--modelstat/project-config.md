---
trigger: always_on
description: Guidance for developers and coding agents in this repo. Keep it current:
---

# AGENTS.md — working on the modelstat daemon

Guidance for developers and coding agents in this repo. Keep it current:
when you change how something here works, update this file in the same PR.

## What this repo is

The modelstat **daemon** — everything that runs on a user's machine and
feeds the server: the Node CLI/daemon (`packages/*`, published to npm +
Homebrew) and the macOS tray app (`apps/tray-mac`). The server
(ingest/pipeline/dashboard, modelstat.ai) is a separate private service
(closed-source) and is out of scope for this repo.

## Naming: daemon, not agent/companion

Our local long-running process is the **daemon** (what `npx modelstat` installs; npm package `modelstat`). Never call it "agent" or "companion".

- **daemon** — our process / CLI / SDK side. (It was historically "companion", and earlier "agent" — both retired.) Use `daemon` for routes (`/v1/daemon/heartbeat`), env (`DAEMON_API_URL`), the launchd label (`ai.modelstat.daemon`), `packages/daemon-core`, the `daemon_version` wire field, etc.
- **agent** — ONLY the user's AI tool (`claude_code`, `cursor`, `codex_cli`, …). Keep "agent" for: the `AGENTS` enum + the `agent` event field, the `/device/:claim/agent` machine-readable view (it's *for* AI agents/LLMs), the `User-Agent` header, this `AGENTS.md`, and "agentic".
- **companion** — retired; don't reintroduce.

## Build & test

```sh
pnpm install
pnpm test          # turbo; per-package: node --import tsx --test src/**/*.test.ts
pnpm typecheck
pnpm build
```

Tests run through the **tsx loader** (`node --import tsx --test`) — do not
switch to `--experimental-strip-types`; it's broken on Node 20 and doesn't
resolve `.js` → `.ts` imports.

Things to know:

- `prices/*.yaml` are deliberate placeholders, not real prices. Don't
  "fix" them and don't write tests that assert specific dollar amounts.
- Parsers (`packages/parsers`) emit raw events only and keep transcript
  data verbatim — e.g. the `<synthetic>` pseudo-model Claude Code records
  for local error/notice messages is passed through as-is (the server
  decides what to hide; the daemon never drops data). The one exception:
  `<synthetic>` must not update the parser's `lastModel` attribution state.
- The summariser is npm-only, single-path (no Ollama, no fallbacks),
  staged via `installNativeRuntime`/`_setup-runtime`.
- Redaction has **one floor**. The secret-pattern catalogue lives in
  `@modelstat/core/redact-floor` (dependency-free) and is the single source of
  truth for the wire redactor (`@modelstat/core/redact`) and the daemon — add a
  newly-leaked credential format there, once. (The standalone `@modelstat/sdk`
  in `sdks/node` keeps its own copy.) The server can *augment* it at runtime via
  a signed, additive `policies` config: the floor always applies and a signed
  bundle can only ever
  add patterns, never remove or weaken them.
- `@modelstat/remote-config` is the shared signed-config loader (fetch → verify
  Ed25519 over raw bytes → disk-cache under `~/.modelstat/config/` → fall back
  memory→disk→bundled). The long-lived daemon refreshes the `policies` kind on a
  timer; new server-delivered config kinds ride this loader instead of forcing a
  release.

## Test fixtures & examples — ALWAYS fictional

This is a **public** repo. Every example, test fixture, sample command, comment
snippet, and doc must be **fictional**. Never paste anything real — not even "a
redaction of" real data, and not even a value you believe is dead or harmless.

- **Secrets** — never a real key/token/password. Synthesize a value that matches
  only the *format* (`sk_live_…`, `1000000000:…`, `phc_…`). The redactor floor
  keys on the variable **name** and length, not the literal value, so a fabricated
  value exercises exactly the same path. Make fakes obviously fake (`examplefake`,
  `EXAMPLE`, sequential `0123…`).
- **Project / app / host names** — use placeholders (`acme`, `globex`,
  `acme-web`, `acme.fly.dev`), never a real repo, Fly app, chain, or any other
  project you (or a user) actually work on.
- **Paths & usernames** — `/Users/dev/Projects/acme`, never a real home dir,
  username, or private-repo layout.

When a parser/redactor bug shows up in real production data, reproduce its
*shape* with a fabricated input — never copy the real string into a test. A
secret committed to a public package or git history can't be unpublished.

**Enforcement.** A `secret-scan` CI job and a gitleaks pre-commit hook (both read
`.gitleaks.toml`) block any real-secret-shaped string; GitHub push protection is
on as well. Fixtures pass because the config allowlists obviously-synthetic
markers by VALUE (`examplefake`, `EXAMPLE`, sequential `0123…`) — never by file
path, so a real secret in a test file still fails. Enable the local hook once
with `pipx install pre-commit && pre-commit install`.

## Releasing (npm + Homebrew)

Releases are **zero-touch**. Every push to main runs `release`
(`.github/workflows/release-daemon.yml`), which decides which publishable packages
changed and what version each gets, then publishes to npm, tags, bumps main,
and cuts a GitHub Release. No `release_type` input, no OTP, no two-phase split.

**How the version is chosen** — `.github/scripts/release-plan.mjs` reads the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modelstat/modelstat](https://github.com/modelstat/modelstat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->

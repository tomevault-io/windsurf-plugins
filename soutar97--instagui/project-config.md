---
trigger: always_on
description: instagui turns any CLI tool into a local web form: capture `<tool> --help`, extract a
---

# CLAUDE.md — instagui project memory

instagui turns any CLI tool into a local web form: capture `<tool> --help`, extract a
Schema with an AI engine, serve a 127.0.0.1-only form that composes and runs the command.

## Architecture
Four folders, strict one-way dependency (ESLint-enforced in `eslint.config.js`, verified by
`test/eslint-dep-rule.test.ts`):
- `shared/` — instagui-agnostic leaf. Imports **nothing** internal (never `core/`/`server/`/`cli/`).
- `core/`   — domain logic. Imports only `shared/` + externals.
- `server/` — HTTP + form. Sits on top.
- `cli/`    — entry point. Sits on top.
- **Schema (Zod) is the single source of truth** (`src/core/schema.ts`): the extraction target,
  the form generator, and the command composer all read the one contract.
- **One validation pipeline** — every engine returns *raw text only*; all of it flows through
  `core/extract.ts`: `Schema.parse` → exactly one retry → on second failure write a debug file
  (`instagui-debug-<tool>-*.json`) and throw. No adapter validates on its own.

## Security invariants — NEVER violate
- **args-array spawn, `shell: false` everywhere.** No shell interpretation of any argv, ever.
  The prompt may ride in argv (`promptVia:'arg'`); a shell would execute metacharacters in it.
  Guarded by `test/engines-cli.test.ts` injection test.
- **Server binds `127.0.0.1` only** (`src/server/server.ts`, NFR-2). Never `0.0.0.0`.
- **CSRF fail-closed Origin check** on state-changing `POST /run` and `POST /stop`: Origin must be
  present AND match the bound host:port, else 403. Read-only routes exempt.
- **No key ever logged, echoed, or served.** Not in diagnostics, not in served pages, not in error
  bodies (OpenAI HTTP errors redact the key from the response). `--engines` prints key *names* only.
- **No raw key on disk.** The config file has no `key` field; keys are read only from the environment
  via `keyEnv`. A config that sets a plaintext `key` is rejected loudly (`config.ts`).
- **Zero new runtime dependencies without a logged decision.** Current budget: `@anthropic-ai/sdk`,
  `zod` — that's it. Adding a third prod dep requires an explicit, recorded decision (license + CVE
  + maintenance check). `package-lock.json` stays committed.

## Engine system
Resolution order (`src/shared/engines/registry.ts` `selectEngine`), first hit wins:
1. `--engine <name>` flag
2. `INSTAGUI_ENGINE` env var (value `claude-code` aliases → `claude`)
3. `default` in `~/.instagui/config.json`
4. auto-detect **API tier**: `anthropic` (ANTHROPIC_API_KEY) → `openai` (OPENAI_API_KEY) →
   `google` (GEMINI_API_KEY) → `deepseek` (DEEPSEEK_API_KEY) — a set key wins over any CLI
5. auto-detect **CLI tier**: `claude` → `codex` → `gemini` (first binary on PATH)
6. none usable → `PreconditionError` (exit 2), actionable message

Eight built-ins (`src/shared/engines/builtins.ts`): `anthropic`, `openai`, `google`, `deepseek`,
`ollama` (all API/local), `claude`, `codex`, `gemini` (subscription CLIs). Three adapter kinds:
`anthropic` (SDK), `openai-compatible` (any `/chat/completions` endpoint), `cli` (spawned tool).
Optional config file `~/.instagui/config.json` (`config.ts`, Zod-validated): `default` + `engines`
map merged **over** built-ins. Absent → empty config. Malformed → PreconditionError. A plaintext
`key` in the file is rejected (use `keyEnv` → env var; instagui never reads a raw key from disk).
`instagui --engines` is the readiness view: per engine it prints whether its key env var is set /
its CLI is on PATH (`describeEngines`) — status only, never a key value.

## Schema resolution order
`--schema <file>` > user cache (`~/.instagui/`) > bundled (shipped: ffmpeg, yt-dlp, pandoc) >
fresh extraction (the only step that hits an engine). `--refresh` re-extracts and overwrites cache.

## Testing rules
- **Hermetic by design** — no real network, DB, time, or credentials. Adapters take injectable
  `run`/`fetchFn`/`onPath`; the suite needs no secrets. Tests requiring a live key self-skip.
- **Golden + hallucination guards** (`src/core/golden.ts`): every extracted flag is re-verified
  verbatim against the locally captured help text. A hallucinated flag fails the build.
- **Hostile-input tests** (`hostile-extraction.test.ts`) + injection/CSRF/Origin negative tests.
- **Full suite green before any commit.** `npm run build && npm run lint && npm test` all pass.

## Plan of record
`docs/2026-07-11-growth-roadmap-and-security-plan.md`. Sequencing:
1. **Security/CI first** (no `.github` exists today): CI matrix (ubuntu+macos+**windows** × Node 22,
   lint→build→test→`npm pack --dry-run`→`npm audit`), npm 2FA, Dependabot (npm + actions, SHA-pinned),
   provenance publishing (`--provenance` via OIDC).
2. **Growth quick wins:** more bundled schemas + demo GIF, SSH remote-browser-launch fix, run
   history/presets, launcher page at `/`.
3. **Flagship (doc labels this v0.2 — see Open decision below):** subcommand trees (git/docker/
   kubectl…) + repeatable flags (ffmpeg `-i`/`-map`) + community schema registry. The registry adds
   a supply-chain surface: **local re-verification of any fetched schema is mandatory, not optional.**

## Conventions
- **Exit codes:** `0` ok · `2` known precondition failure (safe user-facing message, no stack) ·

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Soutar97/instagui](https://github.com/Soutar97/instagui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

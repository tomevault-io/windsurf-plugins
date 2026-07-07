---
trigger: always_on
description: Guidance for Claude Code (and any other AI assistant) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (and any other AI assistant) working in this repository.

## What this project is

icuvisor is an open-source MCP server that connects [intervals.icu](https://intervals.icu) training data to AI assistants, shipped as a single signed Go binary. Read the [PRD](docs/prd/PRD-icuvisor.md) and [ROADMAP.md](ROADMAP.md) before making non-trivial changes — they own scope and phasing.

## Authoritative documents

- **What & why:** [`docs/prd/PRD-icuvisor.md`](docs/prd/PRD-icuvisor.md)
- **When / phasing:** [`ROADMAP.md`](ROADMAP.md)
- **Contributor rules:** [`CONTRIBUTING.md`](CONTRIBUTING.md)
- **Security policy:** [`SECURITY.md`](SECURITY.md)
- **Release process:** [`docs/release/publishing.md`](docs/release/publishing.md)

If these conflict, the PRD wins for product behaviour, the roadmap wins for phasing, and CONTRIBUTING wins for process.

## Hard rules

1. **Clean-room.** This project is MIT-licensed and is built from intervals.icu's public API docs + black-box testing. **Never paste, paraphrase, or transliterate code from any GPL/copyleft source.** Inspiration from public API behaviour is fine; copying source is not.
2. **No GPL dependencies.** Check the license of every new module before adding it. Prefer stdlib first; permissive licenses (MIT, BSD, Apache-2.0, MPL-2.0) second.
3. **Default to `internal/`.** Only put a package in `pkg/` if external consumers genuinely need it.
4. **No `panic` outside `main`.** Return errors. Wrap with `%w`.
5. **Tools must be terse-by-default.** Heavy payloads (streams, raw samples) require an explicit `include_full: true` argument. Token budget is a product feature, not an afterthought.
6. **API keys live in the OS keychain.** Never log them. Never write them to disk in plain text. Never accept them as tool parameters from the LLM — they come from server config, not the conversation.
7. **Don't expose the HTTP transport beyond `127.0.0.1` by default.** A LAN bind is opt-in and must be documented.

## Project layout

```
cmd/icuvisor/            Binary entrypoint (main package only — keep it thin)
internal/app/            CLI dispatch, startup wiring, setup/diagnostics commands
internal/cli/prompt/     Terminal prompting (masked input) for first-run setup
internal/intervals/      intervals.icu HTTP client (typed, tested against fixtures)
internal/mcp/            MCP server wiring + transports (stdio, Streamable HTTP), schema, recovery
internal/tools/          One file per tool, each implementing a small interface
internal/toolcatalog/    Catalog hashing and stale-catalog CI guard surface
internal/toolchecks/     Cross-tool invariants (delete-mode gating, examples, etc.)
internal/coach/          Coach-mode roster, per-athlete tool ACLs, athlete_id routing
internal/safety/         Delete-mode resolution and registration-time gating policy
internal/response/       Terse/full response shaping, scale labels, _meta plumbing
internal/prompts/        Curated MCP prompt registry + golden prompt text tests
internal/resources/      MCP Resources (workout syntax, event categories, athlete profile, schemas)
internal/workoutdoc/     WorkoutDoc Parse/Serialize for the intervals.icu description DSL
internal/credstore/      OS keychain wrapper (macOS Keychain, Windows Cred Mgr, libsecret)
internal/config/         Load/validate/write, athlete-ID/timezone normalization, HTTP bind, dotenv, redaction
internal/diagnostics/    Redacted runtime/config snapshot for `icuvisor diagnostics`
internal/clients/        Shared typed client interfaces (athlete profile, etc.)
internal/units/          Unit enum parsing + preferred-unit conversion
internal/streams/        Canonical stream key normalization
internal/customitemschemas/  Custom-item content schema samples used by write validation
internal/athleteprofile/ Athlete profile read shaping shared by tool + resource
docs/                    PRD, roadmap-adjacent design docs, client setup guides
```

Add new tools as `internal/tools/<tool_name>.go` with a matching `_test.go`. Register them from a single `Register()` call so the catalog is greppable. The registered catalog is the source of truth — the generated website tool reference is regenerated from it (`make docs-tools`) and CI fails on a stale catalog hash.

## Go conventions

- **Format:** `gofmt` + `goimports` with `-local github.com/ricardocabral/icuvisor`. CI fails on dirty diffs.
- **Lint:** `golangci-lint run ./...` must pass. Config: `.golangci.yml`. Don't disable a linter without a comment explaining why.
- **Errors:** sentinel errors for stable contract points (`var ErrNotFound = errors.New("not found")`), wrapped errors everywhere else (`fmt.Errorf("getting activity %s: %w", id, err)`). Use `errors.Is` / `errors.As` at call sites; never `err.Error() == "..."`.
- **Logging:** `log/slog` with structured fields. Use `slog.Default()` in libraries; do not import a global logger. Never log API keys, tokens, or raw athlete identifiers in a way that's hard to scrub later.
- **Context:** every function that does I/O or blocks takes `ctx context.Context` as the first argument. Honour cancellation. No `context.TODO()` in shipped code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ricardocabral/icuvisor](https://github.com/ricardocabral/icuvisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->

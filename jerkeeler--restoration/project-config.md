---
trigger: always_on
description: Onboarding notes for Claude / AI agents and human contributors working in this repo. Keep it tight; update when something here goes stale.
---

# CLAUDE.md

Onboarding notes for Claude / AI agents and human contributors working in this repo. Keep it tight; update when something here goes stale.

## What this is

`restoration` is a CLI that parses Age of Mythology: Retold replay files (`.mythrec`, optionally gzipped as `.mythrec.gz`) into JSON. The replay file is a binary format produced by the game itself; this tool decompresses it, walks its node tree, and decodes the player command log so the data can be inspected, archived, or fed into stats services like aomstats.io.

The format is **not officially documented**. Knowledge of it is reverse-engineered, partly inherited from prior community work (notably loggy's Python proof-of-concept and the next-aom-gg TS parser — see README). This repo does **not** claim to understand every byte of a replay or every command type; see "Known unknowns" below.

## Layout

```
main.go                 # thin entry: cmd.Execute()
cmd/                    # Cobra CLI (root, parse, rename, version)
parser/                 # all replay-decoding logic — see parser/CLAUDE.md
tools/                  # ad-hoc Python probes for patch-debugging (uv-runnable) — see tools/CLAUDE.md
bin/build.sh            # cross-compile for linux/darwin/windows
.github/workflows/      # release workflow, triggers on `v*` tag push
releases/               # build output (gitignored)
```

The CLI is intentionally thin — `cmd/parse.go` validates the path and calls `parser.ParseToJson`. Almost all complexity lives in `parser/`.

## Subcommands

| Command   | What it does                                                |
| --------- | ----------------------------------------------------------- |
| `parse`   | Parse a single replay file → JSON to stdout or `-o` path    |
| `rename`  | Walk a directory and rename replays based on parsed players |
| `version` | Print parser version (sourced from `parser.VERSION`)        |

Global flags: `--is-gzip` (file is gzipped), `-v/--verbose` (debug logging).
`parse` flags: `-o/--output`, `-q/--quiet`, `--pretty-print`, `--slim` (drop game commands), `--stats` (per-player aggregates; mutually exclusive with `--slim`).

The contract for `parse`: **stdout is pure JSON.** All logging goes through `log/slog` to stderr so `restoration parse … | jq …` always works. Don't break this — no `fmt.Println` for diagnostics.

## Build, run, format

```bash
go build -o restoration                  # local dev binary
go run . parse path/to/file.mythrec.gz --is-gzip --slim --pretty-print
bin/build.sh                              # cross-compile all release targets to releases/
go fmt ./...                              # required before commits
go vet ./...                              # recommended sanity check
```

There is no test suite yet (see "Known unknowns"). The release workflow (`.github/workflows/release.yaml`) runs on `v*` tag pushes, builds for linux-amd64 / darwin-amd64 / darwin-arm64 / windows-amd64, and attaches binaries to a GitHub Release.

When bumping the parser version, update `parser/consts.go` (`VERSION`) and tag the commit `vX.Y.Z` to trigger the release.

## Dependencies

- Go 1.23.4
- `github.com/spf13/cobra` — CLI framework

That's the entire external surface. Stdlib handles compression (`compress/gzip`, `compress/zlib`), encoding (`encoding/binary`, `encoding/json`), UTF-16, etc. Keep the dependency footprint small.

## Conventions

- `go fmt` is canonical — tabs, idiomatic Go.
- Use `log/slog` for all logging. Be liberal with `slog.Debug`. Never `fmt.Println` for diagnostics — it leaks into the JSON stream.
- Naming convention in `parser/`: a `parse*` function operates on the raw `[]byte` slice; everything else operates on already-decoded structs. Preserve this split when adding code.
- Errors propagate up to the CLI layer, which prints to stderr and exits non-zero. Don't `os.Exit` from inside the parser.
- Conventional commits in commit messages (e.g. `fix:`, `feat:`, `chore:`).

## Known unknowns (read this before changing the parser)

The replay format is reverse-engineered. Several things are explicitly unfinished:

- **Not all game commands are decoded.** Commands 18, 39, 55, 73, 78 are recognized (their byte length is known well enough to advance the cursor) but their payloads are not extracted. Many other command IDs may exist in the wild and will cause the parser to **fail hard** — see `parser/gameCommandParser.go` where an unregistered command type returns an error with no fallback.
- **Discovering a new command is empirical.** The workflow is: launch the game, perform the action you want to identify, save the replay, diff it against a baseline replay, and locate the new bytes. Hardcoded byte offsets and length heuristics in `parser/gameCommands.go` are the result of this process. Patches to the game can shift these.
- **Team game support is partial.** Winner detection assumes 1v1 (`Winner = !losingTeam`). FFA and team games are not fully validated.
- **No automated tests.** No fixture replays are checked in. Verify changes by parsing real replay files locally and inspecting the JSON diff.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jerkeeler/restoration](https://github.com/jerkeeler/restoration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->

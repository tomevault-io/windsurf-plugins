---
trigger: always_on
description: Orientation for agents working on **machin** (the toolchain) / **MFL** (the
---

# AGENTS.md

Orientation for agents working on **machin** (the toolchain) / **MFL** (the
language). Humans state intent; the machine reads and writes the code.

> **Start here to learn the language: run `machin guide`.** It emits the
> complete, version-exact feature surface — keywords, every builtin with its
> signature, idioms, and gotchas — as JSON (default) or `--text` (prose), from
> the compiler's own source-of-truth catalog (so it never drifts). This file is
> about *contributing to the toolchain*; `machin guide` is about *writing MFL*.

> **First, make sure your `machin` is current.** The guide catalog is compiled
> *into* the binary, so a stale binary advertises a stale feature set — an agent
> running an old `machin` won't see recently added builtins and may wrongly
> conclude a capability is missing (this is what produced #196: `http_get`/`parse`
> already existed, but an out-of-date binary didn't surface them). `make install`
> copies the build to `$PREFIX/bin`, but nothing rebuilds it after a `git pull`,
> so the `machin` on your `PATH` can silently lag the source. Before relying on
> the language surface, rebuild and verify the versions agree:
>
> ```sh
> make install      # or: go build -o "$(command -v machin)" .
> test "$(machin guide | sed -n 's/.*"version": *"\([^"]*\)".*/\1/p' | head -1)" \
>      = "$(sed -n 's/.*machinVersion = "\([^"]*\)".*/\1/p' guide.go)" \
>      && echo "machin is current" || echo "STALE: rebuild/install machin"
> ```
>
> (Also delete any stray `./machin` build artifact in the repo — it is gitignored
> and easy to run by accident.)

## Current direction: dogfood

The POC goal is met; machin is now grown by **building real things** and letting
real use surface the gaps, which then get filled in the language. Recent
examples: a concurrent HTTP health checker added `dial`/`now_ms`/`parse_int`; a
static-site generator added native file I/O (`read_file`/`write_file`/`list_dir`/
`mkdir`) and a parser fix. **When you ship a real app/tool built with machin,
add it to [awesome-machin](https://github.com/javimosch/awesome-machin)** (the
curated ecosystem list). Each app is its own public repo under `javimosch` with
a `build.sh` (`machin encode src/*.src > app.mfl && machin build app.mfl`).

Building a **game** (terminal TUI or raylib GUI/audio)? Read
[`skills/machin-gamedev/SKILL.md`](skills/machin-gamedev/SKILL.md) first — the
canonical setup, build-and-verify workflow, raylib FFI surface, and accumulated
caveats/gotchas (especially MFL's no-implicit-`int`→`float` rule), distilled from
machin-game-snake / -2048 / -flappy / -simon.

## What this is

machin compiles MFL to native code through C:

```
.mfl (canonical text) ──▶ parse ──▶ infer types ──▶ emit C ──▶ cc -O2 ──▶ native binary
```

| Stage | File |
|-------|------|
| Lex / parse | `lexer.go`, `parser.go` |
| Lambda-lift / closure-convert | `transform.go` |
| Type inference + monomorphization | `types.go` |
| C codegen | `codegen.go` |
| Build / run (invokes `cc`) | `build.go` |
| CLI | `main.go` |

The full language reference is [`SPEC.md`](SPEC.md).

## Standing constraints (do not violate)

- **The `.mfl` source of truth is canonical plain text.** One normalized
  function (or type) per line, a blank line between declarations. It is
  greppable, diffable, and editable in place — keep it that way. Machine-first
  means the language is *shaped for machine authoring* (terse, no type
  annotations, canonical one-line form, function-addressable), not that it is
  encoded. A dense base64 "packed" form exists via `machin pack` for
  distribution only; `machin run` reads either form, but the committed source is
  text. (This replaced a base64-as-source design: measured with `tools/tokcost.py`,
  base64 costs an agent ~2.5× the output tokens to write/edit — it taxed the very
  machine-speed it was meant to signal. See PRs/issue history.)
- **Machine-first / minimalism.** Prefer the smallest change that holds the
  surface minimal. The north star is *low agent write/edit cost* (output tokens)
  — measure form/syntax changes with `tools/tokcost.py` and `tools/tokmin.py`,
  don't guess. (Lesson already paid for: tokens are saved by removing what the
  tokenizer *charges* for — whitespace, ~13% — not by shortening keywords it
  already packs into one token; `func`→`fn` saves 0, `println`→`pln` is worse.)
  The canonical `.mfl` form is whitespace-tightened; keep emitted/committed code
  in it (`tighten` in `main.go`, guarded by `TestExamplesAreCanonical`). Target
  C/Rust/Zig-class performance — the default build has no runtime overhead a C
  programmer wouldn't accept.
- Keep the working tree clean. Commit/push only the intended change.

## Dev workflow

```bash
go build ./...                 # build the compiler
go vet ./... && go test ./...  # must be green before a PR
make examples                  # run every example (also: examples/run.sh)
```

Author MFL by writing loose Go-like `.src` text and minting `.mfl` from it:

```bash
go run . encode my.src > my.mfl      # multiple files concatenate (framework + app)
go run . run my.mfl                  # compile to native + execute
go run . run my.mfl --safe           # + bounds / div-zero / overflow checks
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [javimosch/machin](https://github.com/javimosch/machin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->

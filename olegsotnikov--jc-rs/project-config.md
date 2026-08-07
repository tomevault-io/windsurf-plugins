---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

`jc-rs` converts the output of ~236 command-line tools, file formats and strings
to JSON, as a single static binary. It is a Rust implementation of the schemas
defined by [jc](https://github.com/kellyjonbrazil/jc) (Python), pinned as a git
submodule at `./jc` (v1.25.7).

**The product is a compatibility number anyone can check, not speed.**
That premise decides most design arguments:

- Never invent a schema. jc is the authority; where the two disagree, jc-rs has the bug.
- Never edit a fixture to make a test pass. `make check-fixtures` will catch it.
- Never exclude an awkward fixture from the count; report it in a category.
- Publish the number even when it is bad.

Current state: 934/934 = 100% (`tests/differential/REPORT.md`). The workspace
version lives in the root `Cargo.toml`; `./ci/release.sh X.Y.Z` bumps it and
every internal pin in one step.

## Commands

```bash
make build              # cargo build --release
make check              # lint + fixture sync + tests + differential; the universal gate
make lint               # cargo clippy --workspace --all-targets -- -D warnings; cargo fmt --check
make test               # unit + integration tests, pinned to TZ=PST8PDT
make differential       # full jc corpus; rewrites tests/differential/{REPORT.md,report.json}
make bench              # criterion, -p jc-rs-bench
make wasm               # build + Node-test the npm package (needs wasm-pack)
make submodule deps-py  # one-time setup: pin the jc oracle + its optional Python deps
```

Narrower runs:

```bash
TZ=PST8PDT cargo test -p jc-rs-parsers disk::mdadm          # one module's tests
TZ=PST8PDT cargo test -p jc-rs --test integration           # CLI integration tests
python3 tests/differential/validate.py --parser mdadm -v    # differential for one parser
python3 tests/differential/validate.py --fail-under 100     # the CI floor
```

`TZ=PST8PDT` is mandatory and non-obvious: jc's fixtures carry `*_epoch` fields
computed in local time and jc's own `runtests.sh` pins that zone. A bare
`cargo test` produces a pile of meaningless timestamp failures, and a
differential run in another zone silently drops 146 pairs from the denominator.
The Makefile, the CI test job and the differential harness all set it;
hand-run cargo does not.

## The compatibility floor

`--fail-under 100` in `.github/workflows/ci.yml` is what stops the number
sliding. Raise the floor in the same commit that raises the number; never lower
it silently.

## Architecture

Five crates, dependency order `core → utils → parsers → jc-rs`:

| Crate | Role |
|---|---|
| `jc-rs-core` | `Parser`/`StreamingParser`/`LineParser` traits, `ParseOutput`, `ParserInfo`, `ParseError`/`CjError`, and the registry |
| `jc-rs-utils` | shared helpers: `simple_table_parse`/`sparse_table_parse`, `convert_to_*`, `normalize_key`, `parse_timestamp`, `slice_lines` |
| `jc-rs-parsers` | every parser, grouped by domain (`disk/ format/ log/ misc/ network/ package/ proc/ security/ string/ system/`) |
| `jc-rs` | the CLI binary: `args`, `magic`, `meta`, `output`, `streaming` |
| `jc-rs-wasm` | `wasm-bindgen` wrapper: `parse`, `parseRaw`, `parsers`, `StreamSession` |
| `jc-rs-bench` | criterion benchmarks |

**Registration is link-time via `inventory`.** There is no central parser list:
each parser declares a `static INFO: ParserInfo`, a `static X_PARSER`, and an
`inventory::submit! { ParserEntry::new(&X_PARSER) }`. The CLI has
`extern crate jc_rs_parsers;` solely to force linking so those submissions run.
Lookup goes through `find_parser()` (accepts `name`, `kebab-case`, or
`--argument`) and `find_magic_parser()` (matches argv against `magic_commands`).

**Streaming state lives in a session object, not on the parser.** The registry holds
`&'static dyn Parser`, which cannot be downcast, so `Parser::as_streaming()`
hands back the sub-trait and `StreamingParser::session()` mints an owned
`LineParser` carrying the per-run state. A streaming parser's `Parser::parse()`
must be `parse_via_session(self, input, quiet)`. The batch path and the live
path have to be the same code, because the differential only exercises the
batch one.

`docs/api-contracts.md` is the authoritative spec for these interfaces: types,
error-variant semantics, naming conventions (`name` is snake_case, `argument` is
`--kebab-case`), the `_jc_meta` object shape, and exit codes (0 ok, 100 error).
Read it before adding to `jc-rs-core` or writing a parser.

Parser unit tests live in each parser file and `include_str!` fixtures straight
out of `tests/fixtures/`, e.g.
`include_str!("../../../../tests/fixtures/generic/swapon-all-v1.out")` compared
against the sibling `.json`.

## How the compatibility number is produced

`tests/differential/validate.py` walks every `.json` fixture in the pinned jc
submodule and applies one rule: **a fixture pair enters the denominator only when
jc itself reproduces that fixture exactly.** Everything else is reported by
category (`oracle_reject`, `unmapped`, `no_input`) and never dropped.

`tests/fixtures/` is a verbatim mirror of the submodule, enforced by

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OlegSotnikov/jc-rs](https://github.com/OlegSotnikov/jc-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->

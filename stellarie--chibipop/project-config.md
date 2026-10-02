---
trigger: always_on
description: chibipop is a Japanese popup dictionary. A daemon reads screen text with OCR
---

# chibipop — agent operating manual

chibipop is a Japanese popup dictionary. A daemon reads screen text with OCR
and paints a popup. `ARCHITECTURE.md` describes the structure and the rules.
`CONTEXT.md` defines the vocabulary.

## Commands

Linux is the development platform. CI verifies Windows. Every link command
must exclude one bin crate. Both bin crates produce a binary named `chibipop`.
Cargo puts every binary into one target directory. Therefore, a command that
spans both crates causes two linkers to race for one file path.

```bash
cargo build --release -p chibipop-linux
cargo build --release -p chibipop-windows

# The test gates, as ci.yml and release.yml run them.
cargo test --workspace --exclude chibipop-windows   # Linux
cargo test --workspace --exclude chibipop-linux     # Windows

# The OCR quality gate. `test = false` keeps it out of the default sweep.
cargo test -p chibipop-linux --test ocr_gate -- --nocapture

# Japanese analysis unit tests.
cargo test -p chibipop analysis::

# Required after any change under crates/chibipop-linux/src/ocr/.
cargo clippy -p chibipop-linux --test ocr_gate

scripts/package-linux.sh vX.Y.Z
packaging/aur/bump.sh vX.Y.Z

# docs/REGRESSION.md's tiers, by script.
python scripts/manual_regression.py --list
python scripts/manual_regression.py --tier 0 --repo-root . --repeat-tests 3
```

Clippy runs two times across the workspace. You must use `--color never`.
CI sets `CARGO_TERM_COLOR=always`. ANSI escape sequences break the count
anchors if you omit this flag.

```bash
# Pass 1: count lines starting with `warning`, minus cargo's "generated N
# warnings" summaries. Must equal exactly 1. A -D warnings here stops rmeta
# and unlints the dependent bin crate.
cargo clippy --workspace --color never --all-targets --all-features

# Pass 2: count lines starting with `error` or `warning`. Must be 0.
cargo clippy --workspace --color never --all-targets --all-features -- -D warnings \
  -A clippy::while_let_loop -A clippy::doc_lazy_continuation \
  -A clippy::useless_conversion -A clippy::too_many_arguments \
  -A clippy::needless_lifetimes -A clippy::type_complexity
```

## Testing

`docs/REGRESSION.md` defines three tiers.

- **Tier 0** — the automated gate without a screen. CI runs the sweep three
  times to find process-global races. The pass count is a minimum limit, not an
  exact value: 600 for Linux, 400 for Windows.
- **Tier 1** — agent-verifiable on real pixels. `docs/fixtures/ocr-corpus.html`
  publishes its own coordinates.
- **Tier 2** — mostly automatable. It drives the real pointer and hotkeys.

Committed geometry-snapshot goldens verify layout under exact equality. A test
allows no tolerance. Goldens change only through a human-reviewed bless run. A run
of `workflow_dispatch` with `bless=true` rewrites the goldens into an artifact. See
`ARCHITECTURE.md#verification`.

## Project structure

```
src/                     core library `chibipop`: behavior, no OS calls
src/analysis/            Japanese analysis service and model checks
src/select/              Card selection and gesture state
crates/chibipop-linux/   Wayland bin, tray, OCR engine under src/ocr/
crates/chibipop-windows/ Win32 bin, DirectWrite measurement, geometry goldens
docs/                    REFERENCE, LINUX, REGRESSION, RELEASING, BACKLOG
```

`ARCHITECTURE.md` contains the control flow and the rules. `CONTEXT.md`
defines the terms. Do not restate these files.

## Code style

- **Never run a formatter.** See rule 1.
- Render rules as terse lists. Prose explains, but a list decides.
- A comment explains what the code does. Two lines maximum, 50 characters per
  line. It never gives rationale, history, or a rejected alternative. That
  rationale belongs in the pull request.
- No comment on a variable, field, constant, or enum variant unless its meaning
  is not obvious from the name.
- `// SAFETY:` is the one exception. It states the proof that the unsafe block
  needs, at the length that proof needs.
- Write prose in ASD-STE100 Simplified Technical English: one instruction per
  sentence, active voice, and one term for each thing. Code and tables are
  exempt from this rule.

## What never enters the repository

Never commit a reasoning trace, an agent transcript, a research record, a
findings document, or a scratch artifact. Unless oniichan asks for that exact
file, keep it in the working tree and add its directory to `.gitignore`. A
reviewer reads the diff, the tests, and the gate output, not an agent's prose.

## Git workflow

- Write commit messages with Conventional Commits. Use a scope when a scope
  helps: `fix(ocr):`.
- Push work branches to `stellarie/chibipop`. Open pull requests within that
  repository.
- Record the rationale for a decision in the pull request description. This
  repository has no directory for architecture decision records. Never create
  this directory.

## Boundaries

**Always**

- Read the governing section of `ARCHITECTURE.md` before you change behavior.
- Keep the core library free of OS calls, and keep it in physical pixels. Convert
  pixels at the bin seam.
- Pin every dependency in `[workspace.dependencies]`. One pin serves the tree.
- Run the gate of the changed platform. Run the OCR gate when the OCR engine
  changes.

**Ask first**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stellarie/chibipop](https://github.com/stellarie/chibipop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->

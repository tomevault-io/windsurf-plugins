---
trigger: always_on
description: This project uses [`exedocs`](https://github.com/loumtech/exedocs) to keep
---

<!-- EXEDOCS -->
## Executable Documentation (exedocs)

This project uses [`exedocs`](https://github.com/loumtech/exedocs) to keep
documentation in sync with code. Annotated Markdown files under `docs/` are the
source of truth: they generate cram integration tests (`.t` files) and VHS GIFs,
and the pre-commit hook blocks commits that break them.

### Annotation syntax

| Info string | What it produces |
|---|---|
| `` ```sh setup `` | Cram setup line (run before each test block in the `.t` file) |
| `` ```sh gif-setup `` | Shell script sourced in VHS before each recording (for `cd`, env vars, etc.) |
| `` ```sh test `` | Verified command — added to the `.t` cram file |
| `` ```sh test gif="name.gif" `` | Verified command + recorded as a GIF |
| `` ```sh gif="name.gif" `` | Visual demo only — GIF generated, no cram entry |
| `` ```<lang> file="path" [run] [gif="name.gif"] `` | Writes the body to `path`; with `run`, runs it and cram-verifies the paired `output`; with `gif`, records an editor typing animation |
| `` ```output `` | Expected cram output; paired with the preceding `test` or `file` block |

Use `(re)` at the end of an output line to match it as a Python regex (for hashes,
UUIDs, timestamps, etc.):

```
\S+  \S+  \S+  task-1: scaffold auth module (re)
```

**The entire line before `(re)` is interpreted as a Python regex — escape any
special characters:** `\(PM\)`, `\[task-3\]`, `error\.txt`, etc.

Blank lines inside an `output` block are emitted as two-space lines (`  `), which
keeps them inside the cram block. A truly empty line (no spaces) ends the block,
so blank paragraph separators in multi-line output are safe.

**One output-producing command per `sh test` block.** In a multi-command test
block exedocs places the `output` block after the last `$ command` — correct
only when every preceding command is silent (no stdout). If two or more commands
produce output, use a separate `sh test` / `output` pair for each; `exedocs run`
will warn when this constraint is violated.

### File blocks (multi-language consumer tests)

A `` ```<lang> file="path" run `` block writes its body to `path` in the cram
sandbox (via a quoted heredoc, so `$` and backticks are not expanded). Use it to
prove a package works when consumed from another language:

```python file="main.py" run gif="py-edit.gif"
import mypkg
print(mypkg.greet("world"))
```

```output
hello, world
```

- **`file="…"` alone is inert.** A fence with only `file="path"` and no marker
  is treated as a plain filename-labeled snippet (passed through as prose); it is
  not written, not executed, and does not cause exedocs to claim the document.
  Pair it with `run`, `run="cmd"`, or `gif="name.gif"` to activate it.
- `run` executes the file and verifies the paired `output` block. Bare `run`
  infers the command from the extension (`.py`→`python`, `.js`/`.mjs`→`node`,
  `.ts`→`node`, `.rb`→`ruby`, `.php`→`php`, `.pl`→`perl`, `.lua`→`lua`,
  `.sh`→`bash`). For anything else — including compiled languages — give an
  explicit `run="cargo run"` / `run="go run main.go"`.
- exedocs is language-agnostic about **how** the package is available: put
  `pip install`, `npm install`, `cargo add`, or `export VAR=…` in an `sh setup`
  block (cram) and an `sh gif-setup` block (VHS).
- `gif="name.gif"` records an editor typing animation: vim opens the file, types
  the body, `:wq`, then runs it. **VHS cannot type `$`** — a file body
  containing `$` records a broken GIF (cram still passes); drop `gif=` from such
  blocks. One `file=` per block.

### Core workflow

```bash
# 1. Write / edit a doc file with exedocs annotations
# 2. Generate the .t file and any GIFs
exedocs run docs/mycommand.md

# 3. Iterate on test output fast — regenerate .t only, skip GIF recording
exedocs run --no-gif docs/mycommand.md

# 4. Verify the cram tests pass (auto-generates .t if it is missing)
exedocs check docs/mycommand.md

# 5. Register new doc files with the pre-commit hook
exedocs init

# 6. Regenerate GIFs even if hashes haven't changed
exedocs run --force docs/mycommand.md

# 7. Produce a clean rendered Markdown (input first, then -o output path)
exedocs run docs/mycommand.md -o docs/rendered/mycommand.md

# 8. Run the full pre-commit flow by hand (what the generated hook calls)
exedocs hook
```

### Project config (.exedocs.toml, repo root — optional)

```toml
# Run once before any run/check/hook work, so doc tests exercise the code
# being committed rather than a stale installed binary.
prepare = "cargo build --bin mycli -q"
# Prepended to PATH for every cram and VHS subprocess.
path_prepend = ["target/debug"]
```

### Tips

- `sh gif-setup` is VHS-only; `sh setup` is cram-only. Use both when the
  environments need different setup (e.g. cram uses `$TESTDIR`, VHS uses a
  fixed path like `~/demo`).
- VHS `Type` strings cannot contain `$`, so put any shell setup in a
  `gif-setup` block (sourced via a temp script) rather than inline tape commands.
- `exedocs init` automatically adds `**/.exedocs.lock` and `**/*.t` to `.gitignore`
  and is idempotent — run it whenever you add a new annotated file or after a fresh clone.
- If a setup script (e.g. `docs/_setup.sh`) creates demo state for a feature, add

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LoumTechnologies/ticketry](https://github.com/LoumTechnologies/ticketry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->

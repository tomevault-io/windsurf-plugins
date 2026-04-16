---
trigger: always_on
description: This file provides context for AI-assisted maintenance of this repository.
---

# CLAUDE.md — Maintainer Context for dsc011-check-bash

This file provides context for AI-assisted maintenance of this repository.
Read it before making changes to `check_bash`, `test_check_bash.sh`, or
`install_check_bash.sh`. It documents not just what the code does but why
it is written the way it is, including decisions made during development
and bugs caught during pre-release testing.

---

## What This Project Is

`check_bash` is a SHA-256–based answer-checking tool for bash code chunks
in Quarto notebooks. It was built for DSC 011: Computing and Statistical
Programming at UC Merced (instructor: David Ardell, GitHub: dhard).

It is the **bash companion to the R `print_and_check` / `dsc011_check`
system** used in the same course. The design mirrors that system's idioms
as closely as bash allows:

| R system | bash system |
|---|---|
| `answer <- expr` | `MY_CODE='...'` |
| `print_and_check(answer, "hash")` | `eval "$MY_CODE" \| check_bash --code "$MY_CODE" "hash"` |
| `quote()` captures expression once | `MY_CODE` variable used for both eval and `--code` |
| SHA-256 via `digest` package | SHA-256 via `sha256sum` / `shasum -a 256` |

The core design principle: **students write their code expression once**,
and that single expression is both executed and structure-checked.

---

## Target Environment

- **Bash version:** 4.2+ required. Do not attempt to support bash 3.2.
- **macOS:** Ships bash 3.2 (GPL v2; Apple won't update it). Students must
  install bash 5 via `brew install bash` and ensure `/opt/homebrew/bin`
  appears before `/bin` on their PATH. Add `export PATH="/opt/homebrew/bin:$PATH"`
  at the **bottom** of `~/.zshrc` so it wins over Apple's `path_helper`.
- **Windows:** WSL2/Ubuntu ships bash 5.x — no action needed.
- **Shebang:** `#!/usr/bin/env bash` throughout — picks up Homebrew bash 5
  on macOS when PATH is correctly configured.
- **Shell discipline:** `set -euo pipefail` is set at the top of all three
  scripts. See the section below on what this means for arrays and expected
  failures.

---

## Key Design Decisions

### 1. `printf '%s'` not `echo`

`echo` behaves differently across platforms with flags like `-n` and `-e`.
`printf '%s'` is used throughout for portable, predictable output with no
implicit newline. Do not replace `printf '%s' "$var"` with `echo "$var"` —
it will break hash consistency on some platforms because `echo` may add or
interpret escape sequences.

### 2. Dual SHA-256 tool detection

macOS provides `shasum` (Perl); Linux/WSL2 provides `sha256sum` (GNU
coreutils). Both produce identical hashes for the same input. The
`_sha256()` helper auto-detects which is available:

```bash
_sha256() {
    if command -v sha256sum &>/dev/null; then
        sha256sum | awk '{print $1}'
    elif command -v shasum &>/dev/null; then
        shasum -a 256 | awk '{print $1}'
    fi
}
```

Any change to how input is fed into `_sha256()` must preserve hash parity
across platforms. Section 15 of the test suite verifies this against a
known reference value. Cross-platform parity was confirmed during
pre-release testing on Darwin arm64 (bash 5.3.9) and Linux x86_64
(bash 5.2.21), and subsequently verified on both ubuntu-latest and
macos-latest via GitHub Actions CI.

### 3. Empty array guarding under `set -euo pipefail`

Under `set -u`, iterating `"${arr[@]}"` when `arr=()` crashes on bash 3.2
with `unbound variable`. Although we now require bash 4.2+, all four array
iterations in `_run_code_checks()` are guarded with `[[ ${#arr[@]} -gt 0 ]]`
for robustness. This pattern documents the historical failure mode and
prevents regressions if the bash version requirement is ever relaxed:

```bash
if [[ ${#REQUIRES[@]} -gt 0 ]]; then
    for cmd in "${REQUIRES[@]}"; do ...
fi
```

If you add new array arguments, guard them the same way.

### 4. `set -euo pipefail` and expected failures

`set -e` exits on any non-zero return. Commands that are expected to
sometimes fail must be handled explicitly:
- Use `|| true` to suppress expected non-zero exits
- Use `2>/dev/null` to suppress expected error output
- The installer uses both patterns extensively when checking PATH and
  shell rc files

When the installer or `check_bash` exits unexpectedly, a `set -e`
interaction with a failing `grep` or `test` is usually the first thing
to investigate.

### 5. Pipeline counter subshell exclusion

`_count_pipeline_stages()` counts top-level `|` characters only — pipes
inside `$(...)` subshells or quoted strings are excluded. This is
implemented as a character-by-character state machine tracking quote depth
and `$(` nesting depth. Section 9 of the test suite verifies the subshell
exclusion and `||` (logical OR) exclusion.

**Do not simplify this to a `grep -o '|' | wc -l` count** — that would
count pipes inside subshells and break section 9 tests.

### 6. Code analysis is text-only, never executed

The `--code` string is **analyzed as text only** — it is never passed to
`eval`, `bash -c`, or any execution context. The `_strip_comments`,
`_extract_commands`, and `_count_pipeline_stages` functions operate purely
on the string value. Section 13 of the test suite verifies that `$()`,
backticks, and semicolon-chained commands in `--code` strings are not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dhard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->

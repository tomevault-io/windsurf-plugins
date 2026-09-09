---
trigger: always_on
description: Ficus is a self-hosted functional language (arrays first-class; also imperative
---

# Working in the Ficus repo

Ficus is a self-hosted functional language (arrays first-class; also imperative
& OOP) that compiles to portable C/C++. The compiler is written in Ficus and
bootstraps itself.

## Build & run

```sh
make -j8                                  # builds bin/ficus
bin/ficus -run file.fx -- arg1 arg2       # compile + run
bin/ficus -run -O0 -no-openmp -B DIR f.fx # opt level, no OpenMP, build root
bin/ficus -no-c file.fx                   # parse + typecheck only (fast)
bin/ficus -o name file.fx                 # binary at ./name (rm it after),
                                          # intermediates in __fxbuild__/name/
```

- `bin/ficus` finds the stdlib on its own; `-B <dir>` must already exist
  (`mkdir -p` first, or you get a misleading "failed to write some k-forms").
- Diagnostics go to **stdout**, exit code 1. `-run` hides a runtime SIGSEGV as
  exit 1 — to see a real crash, build with `-o` and run the binary directly.
- IR dumps (all work with `-no-c`): `-pr-ast`, `-pr-k0`, `-pr-k` (with `-O3`),
  `-pr-resolve` (overload-resolution trace). Dumps contain every imported
  module; the file under test is the **last** section. Gensyms: `name@1234`.
- `$CC`/`$CXX` pick the C compiler (default `cc`). macOS OpenMP is bundled at
  `runtime/lib/macos_arm64/libomp.a`.
- Build dirs self-invalidate (`.fxstamp`), and fxtest wipes `build/fxtest` on a
  compiler change — no manual `rm -rf` between measurements is ever needed.

**Bootstrap regen** — after editing any `compiler/*.fx` OR any of the 54
bootstrap stdlib modules (`ls compiler/bootstrap/` when in doubt): run
`python3 tools/update_compiler.py`; it copies only changed modules and asserts
the self-hosting **fixpoint** (non-zero exit = determinism regression).
`--check` = dry run; CI runs it on every push touching compiler/lib/runtime —
a stale bootstrap is a red master. This is the only routine self-build of the
compiler, so it alone catches stdlib changes that break compiling the compiler.
Compile-time-only record layout changes (`options_t`, `scope_t`) legitimately
regenerate many bootstrap modules; generated C for *programs* must not change.

## Testing — the fxtest ladder (tools/fxtest/README.md)

```sh
python3 tools/fxtest/fxtest.py all        # unit + negative + ir + cfold + corpus(O0/O3)
bin/ficus -run test/test_all.fx           # the UTest suite directly
```

Layers: T2 corpus differential (O0 vs O3), T3 golden diagnostics
(`test/negative/`), T4 IR snapshots (`test/ir/`), T5 randomized suites
(`test/rand/`). Plus `fxtest.py determinism` and `sanitize` (ASan+UBSan), and the
optional `fxtest.py lsp` leg (builds `tools/FicusLsp.fx` — the Ficus language
server, lsp-1 — and drives it over stdio via `tools/fxtest/lsp_driver.py`;
editor setup + roadmap in `docs/lsp.md`) and the optional `fxtest.py doctut` leg
(doctut-1: `tools/fxtest/doctut.py` extracts every fenced block from
`doc/ficustut.md` and compiles it with `-no-c`, so a syntax reform that breaks a
documented example breaks the leg — nightly in CI, run it after any reform; a
bare fenced block is checked standalone, an HTML comment on the line above
opts out or chains: `<!-- doctut: fragment -->` (deliberately partial),
`<!-- doctut: continue -->` (extends the section's prior blocks), `shell`/`run`).
Both optional legs run **nightly only** to keep the per-PR leg lean.
Harness is Python-3-stdlib-only. Compiler bugs found while on another task are
recorded and fenced (not fixed) in `docs/found_bugs.md`.

**Diagnostics (diag-1).** The type checker recovers and reports MANY errors
per run: a failed definition/arm/branch is reported, its symbol poisoned with
`TypErr` (the declared type when annotated — the annotation firewall), and
`TypErr` operands suppress cascades structurally (`val x = undef; x+1` = ONE
diagnostic). Output: sorted by `(module,line,col)`, primary-line dedup,
`-fmax-errors=N` cap (default 100), Levenshtein `did you mean 'x'?` for
unknown names. Frontend diagnostics (lexer/parser/typecheck/K-normalization,
gated by `Ast.compiler_stage`) carry a gcc-style caret excerpt; middle/backend
ones don't (locations drift). **Every `test/negative/` golden includes the
excerpt**; the T3 harness compares excerpt lines verbatim — use
`--update-golden` when adding cases. New legality checks belong in typecheck
(caret + `-no-c`-visible), not C-gen. **`-diag-format=json` (lsp-1)** emits one
JSON object per diagnostic (jsonl on stdout) instead of the human caret text —
each carries `{file,line0,col0,line1,col1,precision,severity,message,suggestions,
anchor?}`, 1-based (the LSP client converts). Human format is the default and is
byte-identical; every `CompileError` now carries a structured `diag_t` twin, and
json-mode goldens live in `test/negative/25x_json_*` (`// flags: -diag-format=json`).
Stdlib helpers added for the server: `String.utf8_length`/`String.from_utf8`
(byte-exact UTF-8 framing) and `Json.string(js, compact=true)` + proper escaping.

**Spans (reform-prep-1).** `loc_t` is a true span now, not a point: the lexer
returns each batch's `(begin, end)` and the parser stamps every token with it,
so a caret shows the token width (`^~~~`) and `loclist2loc` folds real node
spans. **Gotcha (FB-020): `typ_t` carries NO source location**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vpisarev/ficus](https://github.com/vpisarev/ficus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

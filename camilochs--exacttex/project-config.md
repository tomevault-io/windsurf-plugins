---
trigger: always_on
description: Universal instruction file for AI coding tools (Claude Code, Cursor, Copilot, Aider, OpenCode) and for any
---

# AGENTS.md — ExactTeX

Universal instruction file for AI coding tools (Claude Code, Cursor, Copilot, Aider, OpenCode) and for any
human who wants the five-minute orientation. **Read this before making non-trivial changes.**

Positioning, design rules and the language allowed for claims live in [`PHILOSOPHY.md`](PHILOSOPHY.md). That
file is binding and this one does not restate it — two copies drift and then nobody knows which is current.

---

## 1 · Project intent

ExactTeX gives a writer information about the reliability of their document before they look at the PDF.

It is a one-directional superset of LaTeX: every valid `.tex` is valid input, and a `.xtex` need not compile
under plain TeX. LaTeX stays the backend and the artifact of record.

The two things that justify the project — and the two that must never be traded away — are stated in
`PHILOSOPHY.md` §3: **errors restated in the author's own words**, and **a change model inside the file**.
Faithful byte transport is what makes both reachable from documents that already exist.

---

## 2 · Glossary

- **Entity** — a figure, table, equation, section or citation that the author declared, and therefore that
  the tooling can name. Declared with `@id(x)` on any LaTeX construct, or by a typed block.
- **Light annotation** — `@id(x)` hung off existing LaTeX. Buys checked references and safe rename.
- **Full annotation** — `\figure(x) { … }` with typed fields. Buys visual checks and package synthesis.
- **Opaque region** — source ExactTeX does not model. Transported byte for byte, never rejected. Its type is
  `?O`, the unknown *open* datatype: any package may define new constructors.
- **Coverage** — the fraction of a document that is checked rather than opaque. The analogue of
  `noImplicitAny`. Reported by `xtex check`.
- **Transport** vs **convert** — transport returns the input bytes unchanged; convert produces a different
  artifact. ExactTeX transports. Converters are a one-way trip.
- **Blame** — which side of a boundary a failure belongs to: author LaTeX, a ExactTeX construct, or emitted
  output. A diagnostic without blame is unfinished.

---

## 3 · Repo layout

```
src/
  main.rs  cli.rs  source.rs  lexer.rs
  parser/ { mod.rs, native.rs, latex.rs, raw.rs }
  ast.rs          Span on every node; Opaque node holding raw source
  quarantine.rs   parse-confidence downgrade
  resolve.rs      symbol table, cross-file merge
  types.rs  check.rs
  review.rs       change constructs + sidecar
  emit.rs  sourcemap.rs  texlog.rs
  diagnostics.rs  one model, two renderers (human / JSON)
  bibliography.rs project.rs
  lsp/
tests/  corpus/
```

Directories appear as the tasks that create them land; this is the target shape, not a claim about what
exists today.

---

## 4 · Invariants — never break these

- **Untouched LaTeX comes out byte-identical.** `emit(parse(u)) == u` for input containing no ExactTeX
  constructs. A transporter that sometimes reformats is a false transporter — and the file it corrupts is an already-accepted paper.
- **Annotating never changes the PDF.** Adding a valid annotation must not alter a rendered pixel, and must
  not turn a passing build into a failing one. Test by fuzzing annotations and comparing rasters.
- **Erasure, never injection.** No assertions, wrapper environments or support packages are written into the
  output. Injection breaks the invariant above and collides with packages and catcodes.
- **Opaque bytes are never normalised.** Spans point into immutable source buffers and emission copies the
  original slice. Any emitter "improvement" over an opaque node — reindenting, collapsing whitespace,
  reordering arguments — breaks transport silently.
- **A hard error only ever comes from an explicit ExactTeX construct.** `?O` is consistent with every type, so
  an unannotated `\ref{x}` cannot fail. A renamed `.tex` checks clean by construction. Everything else is
  `advisory` and never touches the exit code. An advisory is printed by default when an explicit construct
  asked for a check that could not be performed, and behind `--strict-tex` when it is only an observation
  about plain LaTeX.
- **Unknown LaTeX is never a fatal error.** The parser downgrades confidence and preserves. Fatal is reserved
  for I/O failure, invalid annotation encoding, resource limits, and broken internal invariants.
- **A diagnostic names its blame side.** Author LaTeX, ExactTeX construct, or emitted output. With no map
  segment to support it, the answer is `unresolved` — never a guess.
- **An entry token must not be able to appear in ordinary LaTeX prose.** Otherwise renaming a `.tex` silently
  changes its meaning.
- **Numbers in docs have a command behind them.** If a README or a PR claims a coverage figure, a runtime or
  an error rate, a documented command reproduces it.
- **Dependencies are permissively licensed.** MIT, Apache-2.0, BSD, ISC. SPDX identifier (the standard machine-readable licence code, e.g. `MIT`, `GPL-3.0-only`) read from package metadata, not recalled. GPL, AGPL and SSPL are never proposed — ExactTeX is MIT, which is why texlab's
  parser cannot be reused.

---

## 5 · Anti-patterns — mistakes already made here


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [camilochs/exacttex](https://github.com/camilochs/exacttex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->

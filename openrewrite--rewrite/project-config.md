---
trigger: always_on
description: This module parses Ruby with **Prism**, run standalone: `RubyParser` hands the source
---

# rewrite-ruby Guidelines

## Parser front end

This module parses Ruby with **Prism**, run standalone: `RubyParser` hands the source
bytes to `org.ruby_lang.prism.wasm.Prism` with explicit `ParsingOptions` and loads the
result with `Loader`. There is no `org.jruby.Ruby` runtime, and the dependencies are
`prism-parser-api`/`prism-parser-wasm` directly, pinned to the versions JRuby 10.1.1.0
ships.

Compile against `org.ruby_lang.prism.*`. The same classes appear relocated as
`org.jruby.internal.prism.*` inside `jruby-complete` — never depend on that jar.

Three things shape `RubyParserVisitor`:

- **Byte offsets.** Every Prism node carries `startOffset`/`length` as offsets into the
  source *bytes*, while the LST and the printer work on the decoded `String`.
  `PrismSource` owns both and translates between them; that translation is what makes
  non-ASCII source round-trip. Use `prefix(node)` to consume up to a node's start — it
  asserts the cursor landed exactly there, so a desync fails with file and offset
  context instead of corrupting everything downstream.
- **No comments, no token positions.** `ParseResult` has no comment array and the node
  classes carry no `nameLoc`/`operatorLoc`/`openingLoc`, so whitespace, comments and
  keywords are still re-lexed from the source with a linear `int cursor`. Node offsets
  anchor that scanning; `peekWhitespace` survives only for genuinely optional tokens
  (`then`, `do`, trailing commas).
- **Heredocs live outside their node's span.** A heredoc node covers only its `<<~ID`
  opener. Openers are queued as they are seen and their bodies are claimed the first
  time the cursor crosses a newline, then folded into the tree by a final pass keyed by
  node id. A body only closes on a line holding nothing but the terminator — indented
  only for `<<~`/`<<-`, at column 0 for a plain `<<ID`. `Rb.Heredoc` keeps the opener and
  the terminator as their own fields, and the printer replays the body at the first
  newline it emits after the opener, so no recipe that rewrites whitespace can strand it.

Three Prism behaviors worth knowing: `partialScript` is on so that fragments with a
top-level `next`/`break`/`return` parse, `mainScript` is off so that a shebang naming
something other than ruby is just a comment, and Prism will not close a heredoc whose
terminator ends the file without a trailing newline, so it is always handed a
newline-terminated copy of the bytes.

Prism models no location for several tokens the printer has to put back, and in each
case the source is read at a node offset rather than guessed:

- **Brackets, parentheses and their absence.** An array literal and an implicit array
  (`a, b = 1, 2`) share one node type, as do a parenthesized target list and a bare
  one; they are told apart by the gap the delimiter leaves between the node's own start
  and its first element. A parameter or argument list is written on the same line as
  the name it follows, so a `(` opening the next line is a grouped expression.
- **Operators and index calls.** `a.+(b)` and `x&.[](i)` are ordinary calls that share
  a node with `a + b` and `x[i]`; what the source writes after the receiver decides.
- **The `=` of an endless method**, found by peeking past the parameter list.
- **The `begin` keyword.** The implicit begin of a `def`, block or class body spans the
  whole construct, so only a node starting at the cursor has a `begin` of its own.

Ruby syntax the visitor has not been taught reaches `defaultVisit`, which throws with
the Prism node name. There are no known gaps: every file that is Ruby in the redmine and
dependabot-core corpora parses. `README.md` has the full inventory of what maps where.

`RubyCorpusTest` measures where a new gap would bite. It is skipped unless
`-Druby.corpus.dir=<dir>` is set, and prints a parse rate plus a histogram of failure
causes; alongside its report it writes `<report>.failures` (one `cause<TAB>path` line
per file) and `<report>.messages` (the full message per file), since the histogram
keeps only one sample per cause.

## Mapping notes

**`;` is a statement separator, and lives on the statement it follows.** A `;` after a
statement is the existing `org.openrewrite.java.marker.Semicolon` marker on that
statement's `JRightPadded`, whose suffix holds the space in front of it — the same
place Java puts it, and the only place that prints in the right order. A `;` with
nothing in front of it (`def x; end`, `if c; body end`, the second `;` of `a;;b`) is a
`J.Empty` statement carrying the same marker, so the statement list still accounts for
every byte and recipes see a real element rather than a hidden one. `bodyStatement`
therefore keeps a `J.Block` whenever a separator is present instead of collapsing to
the sole statement.

**An endless method body is a `J.Block` marked `OmitBraces`** holding one statement:
the block prefix is the space before the `=`, the statement prefix the space after it.
`rewrite-scala` prints `def f = expr` the same way.

**A nested destructuring target is `J.Parentheses` around a bracket-less `Rb.Array`**,
which is the same shape as the top-level target list of `Rb.MultipleAssignment`. `for`
loops instead spread their targets across the names of one `J.VariableDeclarations`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openrewrite/rewrite](https://github.com/openrewrite/rewrite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->

---
trigger: always_on
description: This document explains how to add new language features to the F# parser.
---

# Improving the F# Tree-sitter Parser

This document explains how to add new language features to the F# parser.

## Project Structure

This project contains two parsers:

1. **`fsharp/`** - Parses `.fs` and `.fsx` files (F# source and script files)
2. **`fsharp_signature/`** - Parses `.fsi` files (F# signature files)

**These are not independent.** `fsharp_signature/grammar.js` starts with
`grammar(require("../fsharp/grammar"), { ... })` — it extends the base
grammar and overrides/adds a small number of rules. Consequences you must
internalize:

- A change to `fsharp/grammar.js` changes the *derived* signature grammar
  too, even if you never open `fsharp_signature/grammar.js`.
- Both `fsharp/src/parser.c` **and** `fsharp_signature/src/parser.c` must
  be regenerated after any change to `fsharp/grammar.js`. Forgetting the
  second one will fail CI's "parser matches grammar" check.
- Use `npm run generate` (regenerates both) rather than a single
  `tree-sitter generate` invocation — see the workflow section below.

Each parser directory contains:

- `grammar.js` - The grammar definition (written in JavaScript)
- `src/grammar.json` - Generated grammar in JSON format
- `src/parser.c` - Generated C parser code (never edit this file directly)
- `src/scanner.c` - Thin per-parser shim that `#include`s the shared scanner

The **real** external scanner logic lives in a single shared header at
[`common/scanner.h`](./common/scanner.h). Both `fsharp/src/scanner.c` and
`fsharp_signature/src/scanner.c` are just three-function shims that delegate
to functions defined in that header. If you need to change tokenization
behavior (indent/dedent, keyword handling, string interpolation, etc.),
edit `common/scanner.h` — never the per-parser `scanner.c` shim.

## Running the CLI

The `tree-sitter` CLI is installed as a local devDependency
(`node_modules/.bin/tree-sitter`), not globally. Invoke it with
`npx tree-sitter <command>`, which resolves the local binary
automatically. All examples in this document use `npx tree-sitter ...`.

## Workflow for Adding a New Feature

### 1. Create a Test Case First

Before implementing anything, write a test case in the appropriate corpus file under `test/corpus/`. Tests for `fsharp` go in `test/corpus/*.txt`, and tests for `fsharp_signature` go in `test/corpus/fsharp_signature/*.txt`.

Test format (from [tree-sitter docs](https://tree-sitter.github.io/tree-sitter/creating-parsers/index.html#the-test-format)):

```
================================================================================
test name
================================================================================

<input code>

--------------------------------------------------------------------------------

<expected parse tree>
```

Example from `test/corpus/constants.txt`:

```
================================================================================
simple string
================================================================================

let x = "test"

--------------------------------------------------------------------------------

(file
  (declaration_expression
    (function_or_value_defn
      (value_declaration_left
        (identifier_pattern
          (long_identifier_or_op
            (identifier))))
      (const
        (string)))))
```

To define the expected parse tree you should use the following fsi script to obtain the fsharp AST.
Translate the AST into the lisp like format ued by tree-sitter. Refer to the grammar.json for the correct node names and structure.
If the appropiate node names are not present in the grammar, you may need to update grammar.js to add the necessary rules and regenerate the parser before you can write the test.

```fsi
#r "nuget: Fantomas.FCS, 7.0.5"
let sample = ""
Parse.parseFile false (SourceText.ofString sample) []
```

where sample if the code you want to parse.

### 2. Run the Test

From the repo root (so `tree-sitter.json` picks up both parsers):

```bash
npx tree-sitter test
```

To limit to a single test file or a single test name:

```bash
npx tree-sitter test --file-name attributes.txt
npx tree-sitter test -i "top-level module attribute"
npx tree-sitter test --overview-only   # pass/fail summary only
```

To parse a single file (useful for iterating on real F# samples):

```bash
npx tree-sitter parse path/to/file.fs
# .fsi files need the signature parser selected explicitly:
npx tree-sitter parse -p fsharp_signature path/to/file.fsi
```

### 3. Implement the Feature

Update `grammar.js` in the appropriate parser directory to add the new grammar rule. If a feature applies to both `.fs` and `.fsi`, update **both** `fsharp/grammar.js` and `fsharp_signature/grammar.js`.

If you need to add special tokenization logic (indent/dedent behavior, keyword handling that competes with identifiers, string interpolation, etc.), edit the shared scanner at `common/scanner.h` — not the per-parser `src/scanner.c` shim.

### 4. Regenerate the Parser(s)

After modifying `grammar.js`, regenerate `grammar.json` and `parser.c` for
**both** parsers. The safe default is:

```bash
npm run generate
```

This runs `tree-sitter generate` inside each parser directory and produces
both `fsharp/src/{grammar.json,parser.c}` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ionide/tree-sitter-fsharp](https://github.com/ionide/tree-sitter-fsharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

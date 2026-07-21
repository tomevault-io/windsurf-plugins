---
trigger: always_on
description: `mx` is a MusicXML C++ library. The product surface is `mx::api`, a simplified and narrowed
---

# mx

@./README.md

`mx` is a MusicXML C++ library. The product surface is `mx::api`, a simplified and narrowed
interface of MusicXML backed by the strongly-typed `mx::core` model.

Quickstart: `make core-roundtrip-test` (fast corert gate) - `make fmt` (format) - `make help` (all
targets). Everything runs in the `mx-sdk` Docker image unless `MX_RUNNING_IN_DOCKER=1`.

## Repository layout

```
mx/
  AGENTS.md             <- you are here
  .claude/skills/       <- agent skills; every skill name starts with mx-
  Makefile              <- top-level build driver
  Dockerfile            <- mx-sdk image with toolchains and dev tools
  CMakeLists.txt        <- C++ project
  Package.swift         <- Swift Package Manager wrapper (built by the swift CI job)
  data/                 <- MusicXML test corpus (large, see data/README.md)
  docs/ai/design/       <- design docs
  docs/musicxml-4.0-ed15c23.xsd <- the current musicxml.xsd schema
  src/                  <- C++ sourcecode root
  src/include/mx/api/   <- the public mx::api headers; mx's public interface
  src/private/          <- C++ source
    pugixml/            <- vendored pugixml 1.15 (XML parser and raw DOM)
    mx/core/            <- mx::core's hand-written runtime (outside of mx/core/generated)
    mx/core/generated/  <- MusicXML XSD C++ typed model (generated, see gen/cpp)
    mx/api/             <- mx::api implementation
    mx/impl/            <- the api to core translation layer
    mx/utility/         <- helpers
    mx/examples/        <- api example programs
    mxtest/core/        <- mx::core unit tests (small)
    mxtest/corert/      <- C++ core roundtrip test (deserializes and serializes the test corpus)
    mxtest/import/      <- normalization support (Normalize.cpp/h + DecimalFields.h) shared by corert and api tests
    mxtest/api/         <- tests for the mx::api layer
    mxtest/impl/        <- tests for the mx::impl layer
    mxtest/file/        <- file-walking test infrastructure for the mx::api tests (MxFile, MxFileTest, etc.)
    mxtest/control/     <- compile-control flags (CompileControl.h: enables/disables test suite compilation)
    cpul/               <- vendored Catch2 test runner
  gen/                  <- code generator system (see gen/README.md)
    cpp/                <- C++ target (the product): config.toml + templates
    schema/             <- JSON Schema target
    test/go/            <- A toy Go implementation of MusicXML XSD for gen validation
    test/c/             <- A toy C implementation of MusicXML XSD for gen validation
  audit/                <- MusicXML feature-audit tool (see audit/README.md); `make audit`
```

## Feature audit (`audit/`)

`make audit` inventories corpus feature usage (`*.features.xml` sidecars + `data/corpus.xml`,
checked in; the test suites skip them). See `audit/README.md`; the `mx-api-feature-audit` skill
consumes the output.

## Build system

The `Makefile` is the entrypoint for all build processes; it drives `cmake` and runs everything
inside the `mx-sdk` Docker image unless `MX_RUNNING_IN_DOCKER=1` (see the README's Build section).
Lean on it and suggest improvements when it doesn't have what you need. Run `make help` for the
target list. Extend the `mx-sdk` image when new tools are needed.

## The corert (core roundtrip) test

Runs with `make core-roundtrip-test`. Deserializes and reserializes the test corpus with `mx::core`. It
compiles without `mx::impl`/`mx::api`, so you can iterate on `gen/cpp` and the generated core and
defer fixing the `mx::impl` integration until you are ready.

### Flow (same in all three languages)

1. **Discover** eligible `.xml`/`.musicxml` files under `data/` (excluding certain directories and
   marker files; see `data/README.md`). Files with a sibling `.invalid` marker are skipped as
   unparseable.
2. For each file: Load the XML into a DOM, make certain expected alterations, parse it with
   `mx::core`, serialize it back to XML, normalize the output, and compare the two DOMs.
3. Report pass/fail per file.

## Generator (`gen/`)

`gen/` is a Python code generator (`python3 -m gen`) that reads the MusicXML XSD and emits typed
serialization/deserialization libraries. See:
- `gen/AGENTS.md`: your entrypoint
- `gen/DESIGN.md`: how it was designed
- `gen/README.md`: human summary

What you need to know right now is that `gen/cpp` is where our MusicXML types are coming from. Run
`make gen-cpp` to regenerate the C++ types.

## C++ coding rules

Do not add new anonymous namespaces (`namespace { }`); many existing files still have them and
they are being retired as touched. Anonymous namespaces give internal linkage, which is correct for a normal
one-TU-per-file build but causes redefinition errors in unity builds (where multiple `.cpp` files
are compiled as a single translation unit). Use a named helper function or per-type name instead:

- For file-local helper functions: name them after the type or file, e.g. `tokenIsNameChar`,
  `clampedTenths`.
- For file-local constants (arrays, string_views): use a per-type prefix, e.g. `kYesNoWire`,
  `kSmuflAccidentalGlyphNamePrefix`.
- In code generator templates (`gen/cpp/templates/`): use `{{ident}}` to make names unique, e.g.
  `k{{ident}}Wire`, `isNameChar{{ident}}`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webern/mx](https://github.com/webern/mx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

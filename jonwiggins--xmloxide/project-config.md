---
trigger: always_on
description: **xmloxide** is a pure Rust reimplementation of libxml2 — the de facto standard XML/HTML parsing library in the open-source world. libxml2 became officially unmaintained in December 2025 with known security issues. xmloxide is a memory-safe, high-performance replacement that passes the same conformance test suites.
---

# CLAUDE.md — xmloxide project guide

## Project Overview

**xmloxide** is a pure Rust reimplementation of libxml2 — the de facto standard XML/HTML parsing library in the open-source world. libxml2 became officially unmaintained in December 2025 with known security issues. xmloxide is a memory-safe, high-performance replacement that passes the same conformance test suites.

**Version:** 0.4.1
**License:** MIT
**MSRV:** Rust 1.81+

### Achieved Goals

- Full conformance with W3C XML 1.0 (Fifth Edition) and Namespaces in XML 1.0
- **1727/1727** W3C XML Conformance Test Suite tests passing (100%)
- **119/119** libxml2 regression tests passing (100%)
- Feature parity with libxml2's core: XML/HTML parsing, DOM, SAX2, XPath 1.0+, XmlReader, push parser, DTD/RelaxNG/XSD/Schematron validation, C14N, XInclude, XML Catalogs
- **WHATWG HTML5 parser** — full HTML Living Standard tokenizer (§13.2.5) and tree builder (§13.2.6) with 7032/7032 tokenizer tests + 1778/1778 tree construction tests passing (100% html5lib-tests)
- **CSS selector engine** — query elements with familiar CSS syntax including combinators and pseudo-classes
- **Serde integration** — optional XML (de)serialization to/from Rust types via serde
- **Async parsing** — optional async parsing from `tokio::io::AsyncRead` sources
- Zero `unsafe` in public API surface (`unsafe_code = "deny"` in Cargo.toml)
- No system dependencies — pure Rust (uses `encoding_rs` for character encoding)
- C/C++ FFI layer with header file (`include/xmloxide.h`)
- WASM bindings (`xmloxide-wasm`) and Python bindings (`pyxmloxide`)
- `xmllint` CLI tool
- Performance competitive with libxml2 (serialization 1.5-2.3x faster)

### Non-Goals

- XSLT (that's libxslt, a separate project)
- XML 1.1 support (rarely used, can add later)
- Full libxml2 API-level compatibility (we design an idiomatic Rust API)

---

## Architecture

### Tree Representation (the critical design decision)

libxml2 uses a web of raw C pointers (parent, children, next, prev, doc, ns). We use **arena allocation with typed indices**:

- `Document` owns a `Vec<NodeData>` — all nodes live here
- `NodeId` is a `#[repr(transparent)]` newtype over a `NonZeroU32` index
- Navigation (parent, first_child, next_sibling, prev_sibling) stored as `Option<NodeId>` fields on each `NodeData`
- This gives us O(1) node access, cache-friendly layout, no reference counting overhead, no borrow checker fights, and safe freeing (drop the Document, everything is freed)
- Trade-off: individual node removal requires a free-list or tombstone approach

**Why not `Rc<RefCell<>>`:** Reference cycles between parent/child require weak refs, runtime borrow panics are possible, and per-node allocation is slow.

**Why not raw pointers behind unsafe:** We want to minimize unsafe surface area. Arena indices give us the same performance with compile-time safety.

### Module Map

```
src/
├── lib.rs              # Public API re-exports, crate docs
├── bin/
│   └── xmllint.rs      # xmllint CLI (behind "cli" feature)
├── tree/
│   ├── mod.rs          # Document, NodeId, NodeData, tree navigation, id_map
│   └── node.rs         # NodeKind enum (element/text/comment/PI/doctype/etc)
├── parser/
│   ├── mod.rs          # ParseOptions, top-level parse functions
│   ├── xml.rs          # XML 1.0 parser (core state machine)
│   ├── push.rs         # Push/incremental parser wrapper
│   └── input.rs        # Parser input stack (entity expansion, includes)
├── html/
│   ├── mod.rs          # HTML 4.01 parser (error-tolerant, auto-closing, void elements)
│   └── entities.rs     # HTML named character references
├── html5/
│   ├── mod.rs          # WHATWG HTML5 parser public API and module docs
│   ├── tokenizer.rs    # HTML5 tokenizer (all 80 states per §13.2.5)
│   ├── tree_builder.rs # HTML5 tree construction (all insertion modes per §13.2.6)
│   ├── sax.rs          # HTML5 SAX-like streaming API (no DOM tree)
│   └── entities.rs     # HTML5 named character references (§13.5, 2231 entries)
├── css/
│   ├── mod.rs          # CSS selector public API (select, select_with)
│   ├── parser.rs       # CSS selector parser
│   ├── types.rs        # Selector AST types
│   └── eval.rs         # Selector evaluator against document trees
├── sax/
│   └── mod.rs          # SAX2 handler trait and streaming parser
├── reader/
│   └── mod.rs          # XmlReader pull-based parsing API
├── encoding/
│   └── mod.rs          # Encoding detection, BOM sniffing, encoding_rs bridge
├── xpath/
│   ├── mod.rs          # XPath public API, evaluate() entry point
│   ├── ast.rs          # XPath AST types (Expr, Step, Axis, etc.)
│   ├── lexer.rs        # XPath expression tokenizer
│   ├── parser.rs       # XPath expression parser → AST
│   ├── eval.rs         # Expression evaluator against a node tree
│   ├── types.rs        # XPath value types (NodeSet, String, Number, Boolean)
│   └── regex.rs        # XPath regex support (matches, replace, tokenize)
├── validation/
│   ├── mod.rs          # ValidationResult, ValidationError

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonwiggins/xmloxide](https://github.com/jonwiggins/xmloxide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

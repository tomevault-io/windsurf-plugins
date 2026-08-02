---
trigger: always_on
description: <!-- Agent-consumed file. Keep terse, unambiguous, machine-parseable. -->
---

<!-- Agent-consumed file. Keep terse, unambiguous, machine-parseable. -->

# Helium

XML toolkit for Go covering XML parsing, SAX2-style streaming, XPath 3.1,
XInclude, XSD, Relax NG, and Schematron. Started as a libxml2-style port to
Go and grew broader native Go APIs and features along the way.

## XPath 3.1 — XSD Version

The xpath3 package targets **XSD 1.1 only**. This means `+INF` is a valid lexical form for xs:double and xs:float, and xs:dateTimeStamp is a recognized type. QT3 tests with `dependency type="xsd-version" value="1.0"` are skipped.

## XSD — Version Toggle

The xsd package defaults to **XSD 1.0** and treats 1.1 as **opt-in** via `Compiler.Version(xsd.Version11)` (or a `vc:minVersion="1.1"` hint on the root `<xs:schema>` when no explicit version is set). The resolved version is frozen onto the compiled `Schema` so the `Validator` applies the same semantics. 1.0 stays the default so existing behavior and goldens are unchanged.

`resolveVersion` (`compile.go`) resolves in order: a forced `Compiler.Version()` (always wins) → a `vc:minVersion="1.1"`-or-higher hint on the root → a configured `Compiler.DefaultVersion(v)` (the opt-in fallback for a schema silent on version) → `Version10`. `DefaultVersion` never overrides a forced version or a vc hint — it only chooses the fallback. The STANDALONE default stays `Version10`; `DefaultVersion` lets an embedding layer opt its schemas into 1.1 by default while still honoring an explicit version.

`Validator.SkipDatatypeIntegrityChecks(true)` suppresses the document-wide datatype-integrity walks in `validateDocument` (`cfg.skipDatatypeIntegrity`): the xs:ID/xs:IDREF/xs:IDREFS uniqueness+referential-integrity walk (version-INDEPENDENT — runs in both 1.0 and 1.1) and the XSD 1.1-only xs:ENTITY/xs:ENTITIES walk; content-model, type, and xs:key/unique/keyref identity-constraint validation are unaffected. It is for callers that validate an element/subtree as a fragment and enforce document-scope ID/IDREF integrity themselves (xslt3). In 1.0 it suppresses the ID/IDREF walk (the ENTITY walk never runs there).

XSD 1.1 is fully implemented behind the `Version11` opt-in (967/0 on the W3C suite). The complete feature-by-feature implementation state — every 1.1 construct, its file/function, spec clause, version gating, W3C test evidence, and remaining gaps — lives in `.claude/docs/xsd11.md`. **Read that doc before any work in `xsd/`.** Feature areas covered there:

- Type system: xs:assert (complex + simpleContent), xs:assertion facet, conditional type assignment (xs:alternative), simpleContent content-type narrowing, attribute inheritance, 1.1 built-in datatypes, simple-type 1.1 edges.
- Content models: UPA weakening, open content (xs:openContent / xs:defaultOpenContent), xs:all relaxations, wildcard notNamespace/notQName, particle-restriction relaxations, content-model backtracking, Wildcard EDC.
- Identity constraints: field-node classification/canonicalization, @ref, @xpathDefaultNamespace, structural rules, skip-wildcard scoping.
- Document-wide walks: xs:ID/IDREF/IDREFS and xs:ENTITY/ENTITIES integrity.
- Schema composition & representation: xs:override, xsi: attribute references, conditional inclusion (vc:), NCName/QName whitespace collapse, xs:notation, and the many version-INDEPENDENT XML-representation/structural checks.

Do NOT enforce 1.1-only clauses in the 1.0/default path — 1.0 must stay byte-identical to origin.

## XSLT 3.0 — Conformance Scope

The xslt3 package targets **Basic XSLT 3.0** conformance (W3C spec Section 27). The spec defines 8 conformance levels; only "Basic XSLT Processor" is required. The remaining 7 are optional features:

| Feature | Status | Notes |
|---------|--------|-------|
| Basic XSLT Processor | **Target** | Core requirement |
| Schema-Awareness | In progress | `xsl:import-schema`, type annotations |
| Serialization | Implemented | xml/html/text output methods |
| Streaming | Implemented | DOM-materialization; XTSE3430 analysis |
| Higher-Order Functions | Implemented | Via xpath3 |
| XPath 3.1 | Implemented | Via xpath3 |
| Dynamic Evaluation | Implemented | `xsl:evaluate` |
| Backwards-Compatible Processing | Implemented | XSLT 1.0 behavior + XPath 1.0 compatibility mode |

Schemas imported via `xsl:import-schema` (and a source-document schema) default to **XSD 1.1** (`compile_schema.go`/`source_schema.go` build the `xsd.Compiler` with `.DefaultVersion(xsd.Version11)`), so a schema silent on version compiles with 1.1 semantics (CTA/xs:alternative, unions, etc.) while an explicit `Compiler.Version()` or a schema `vc:minVersion` hint still wins. xslt3 validates a constructed element/subtree through `schemaRegistry.ValidateDoc` with `xsd.Validator.SkipDatatypeIntegrityChecks(true)`: content/type/CTA validation runs at 1.1 but the XSD 1.1 document-wide xs:ID/IDREF/ENTITY integrity walks are suppressed, because element-level validation (`xsl:validation="strict"` on an LRE) must not enforce whole-document ID uniqueness — xslt3 applies document-scope ID/IDREF integrity itself via `validateDocIDConstraints` at the true document/result-document scope (XTTE1555), matching the W3C validation-16xx semantics.

## XSLT — Backwards-Compatible Processing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lestrrat-go/helium](https://github.com/lestrrat-go/helium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

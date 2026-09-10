---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Behavioral

1. Don’t assume. Don’t hide confusion. Surface tradeoffs.
2. Minimum code that solves the problem. Limit speculative additions.
3. Touch only what you must, clean up only your own mess -- but do suggest additional related fixes.
4. Define success criteria. Loop until verified.

## Project Overview

Woodstox is a high-performance Java XML processor implementing the StAX (Streaming API for XML, `javax.xml.stream`), SAX, and extended Stax2 APIs. It optionally supports XML Schema and RelaxNG validation via MSV (Multi-Schema Validator). The library targets Java 8+ and produces an OSGi bundle JAR with shaded MSV dependencies.

Maven coordinates: `com.fasterxml.woodstox:woodstox-core` (Java package root is the legacy `com.ctc.wstx`).

## Build Commands

```bash
./mvnw clean install              # Build and run tests
./mvnw clean install -DskipTests  # Build without tests
./mvnw test                       # Run all tests
./mvnw test -Dtest=TestClassName  # Run a single test class
./mvnw test -Dtest=TestClassName#methodName  # Run a single test method
```

CI (`.github/workflows/main.yml`) tests against Java 8, 17, and 21 using `./mvnw -B -q -ff -ntp verify`; the Java 8 run is the "release build" that also produces JaCoCo coverage. There is no separate lint step.

Note that `package` runs shading + moditect, so `install`/`verify` exercise more than `test` does — a change that compiles and tests fine can still break the bundle/module step.

## Architecture

All main source is under `src/main/java/com/ctc/wstx/`.

### Reading pipeline

`WstxInputFactory` (in `stax/`) is the entry point; all of its `createXMLStreamReader`/`createXMLEventReader` overloads funnel into private `createSR(...)` methods. Those pick a *bootstrapper* (`io/StreamBootstrapper` for byte streams, `io/ReaderBootstrapper` for `Reader`s) which sniffs/validates the XML declaration and encoding, then construct the reader.

The reader class chain is a linear inheritance stack, each layer adding one concern:

```
StreamScanner            (sr/)  low-level tokenizing, entity expansion, input source stack
  └─ BasicStreamReader   (sr/)  the ~5800-line core: StAX event loop and state machine
      └─ TypedStreamReader (sr/) Stax2 typed access (getElementAsInt etc.)
          └─ ValidatingStreamReader (sr/)  validation hookup — this is what gets instantiated
```

Element/namespace/attribute state lives beside the reader in `InputElementStack` and `AttributeCollector` rather than in the reader itself; `InputElementStack` is also where validators are attached (`validateAgainst()`).

`io/` holds the encoding-specific readers (`UTF8Reader`, `UTF32Reader`, `AsciiReader`, `ISOLatinReader`, `EBCDICCodec`) and input-source plumbing (`WstxInputSource`, `BranchingReaderSource` for DTD-internal-subset capture, `DefaultInputResolver` for external entities). Bug fixes for malformed input (overlong UTF-8, out-of-range code points, bad `EncName`) usually land here or in the bootstrappers.

### Writing pipeline

`WstxOutputFactory.createSW(...)` builds a *two-layer* stack, which is the main thing to understand about `sw/`:

1. A low-level `XmlWriter` that owns encoding and character escaping. Which subclass depends on the target encoding: `AsciiXmlWriter`, `ISOLatin1XmlWriter` (both extending `EncodingXmlWriter`, byte-backed) or `BufferingXmlWriter` (char/`Writer`-backed).
2. An `XMLStreamWriter2` implementation on top, chosen by namespace configuration: `NonNsStreamWriter` (no namespaces), `SimpleNsStreamWriter` (namespaces, caller-supplied prefixes), or `RepairingNsStreamWriter` (auto-repairing). The latter two extend `BaseNsStreamWriter`; all extend `TypedStreamWriter` → `BaseStreamWriter`.

A consequence worth remembering: escaping/well-formedness bugs often must be fixed in *both* the byte-backed (`EncodingXmlWriter` subclasses) and char-backed (`BufferingXmlWriter`) writers, and tests should cover both. Output element/namespace state lives in `SimpleOutputElement`/`OutputElementBase`.

### Validation

Validation is pluggable through the Stax2 `XMLValidationSchema` / `XMLValidator` abstraction, so both DTD and MSV-based validation attach to readers and writers the same way.

- `dtd/` (largest package) is a self-contained DTD implementation: `FullDTDReader`/`MinimalDTDReader` parse subsets into a `DTDSubset` of `DTDElement`s and `DTDAttribute` subclasses (one per attribute type); content models compile to DFAs (`DFAState`, `DFAValidator`, `StructValidator`). `DTDValidator`/`DTDValidatorBase` is the runtime validator, `DTDSchemaFactory` the Stax2 entry point.
- `msv/` is a thin bridge to MSV for W3C Schema and RelaxNG: `W3CSchemaFactory`/`RelaxNGSchemaFactory` produce schemas whose validator is `GenericMsvValidator`.

### Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FasterXML/woodstox](https://github.com/FasterXML/woodstox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

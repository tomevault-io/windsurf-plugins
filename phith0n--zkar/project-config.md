---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ZKar is a pure-Go parser, viewer, and manipulator for the Java Object Serialization protocol (the `0xACED 0x0005` stream produced by `ObjectOutputStream`). No CGO or JDK is required. A `class` package for parsing `.class` bytecode, an `rmi` package for parsing the JRMP wire protocol, and a CLI (`main.go`) are also part of the repo. The project ships as both a library (`github.com/phith0n/zkar/serz`, `.../class`, `.../rmi`, `.../commons`) and a CLI binary released via goreleaser.

## Common commands

```shell
# Run the CLI
go run main.go dump -f testcases/ysoserial/CommonsCollections6.ser
go run main.go dump -B <base64-payload>
go run main.go dump --jdk8u20 -f testcases/pwntester/JDK8u20.ser   # see JDK8u20 note below
go run main.go dump --golang -f <file>                             # emits a Go literal via litter
go run main.go rmi  -f <jrmp-capture.bin>                          # parses JRMP framing; see rmi/ section

# Full test suite (also prints the README's gadget table to stdout as a side effect of TestMain)
go test -v ./...
go test -race ./...

# Single test / package
go test ./serz -run TestYsoserial -v
go test ./serz -run TestCC6WithOverlongEncoding -v

# Lint (CI uses golangci-lint latest; config in .golangci.yml)
golangci-lint run --verbose
```

CI matrix runs on Go 1.18–1.22 across Linux/macOS/Windows; keep code compatible with Go 1.18 (module `go 1.18`). Release tags `v*` trigger goreleaser.

## Architecture

### `serz` — the Java serialization codec

Every Java serialization stream is a magic number + version + a list of `TCContent` records. The parser is a recursive-descent reader over a byte stream; every on-wire grammar element has a Go type whose name matches the Java spec (`TCObject`, `TCClassDesc`, `TCProxyClassDesc`, `TCArray`, `TCString`, `TCEnum`, `TCReference`, `TCNull`, `TCBlockData`, `TCClass`, `TCFieldDesc`, `TCValue`, `TCUtf`, `TCClassPointer`, `TCStringPointer`). Follow that naming when adding grammar.

Key contracts:

- **`Object` interface** (`serz/model.go`-adjacent, defined in `parser.go`): every grammar node implements `ToBytes() []byte`, `ToString() string`, and `Walk(WalkCallback) error`. `ToBytes` must be a faithful round-trip of what was parsed — the test suite asserts `bytes.Equal(original, ser.ToBytes())` for every ysoserial fixture. Do not "normalize" output.
- **`TCContent`** (`serz/tc_content.go`) is the tag-dispatcher. Its `Flag` field selects which of the sibling pointers is populated, and `ToBytes`/`ToString`/`Walk`/`ReadTCContent` all fan out via the same `switch` on `Flag`. When you add a new `JAVA_TC_*` tag you must extend all four switch statements plus `serz/model.go` constants.
- **`ObjectStream`** (`serz/buffer.go`) wraps `commons.Stream` and owns the handler table. Every referenceable object (object, class, class desc, proxy class desc, string, array, enum) must call `stream.AddReference(obj)` at the point Java would assign it a wire handle (starting at `JAVA_BASE_WRITE_HANDLE = 0x7e0000`). `TCReference` resolves back via `GetReference`. Mis-ordering `AddReference` calls silently corrupts payloads — order them exactly where the Java writer would.
- **`Walk` / `FindObject` / `FindClassDesc`** (`serz/walker.go`) provide a generic visitor over the tree. `FindObject` aborts traversal by returning a sentinel `*StopWalkError`; preserve that pattern for short-circuit searches. `TestCC6WithOverlongEncoding` is the canonical example of mutating a parsed tree and re-serializing.
- **`serz/builder.go`** holds hand-written constructors (`NewTCString`, `SimpleClassDesc`, `NewTCValueBytes`, …) for programmatically building payloads. Prefer extending these over ad-hoc struct literals at call sites.
- **`DumpToGoStruct`** (`serz/go-dumper.go`) renders a parsed tree as Go source using a forked `litter`. `[]*TCValue` byte arrays are special-cased to emit `zkar.NewTCValueBytes([]byte("\x..."))` — keep that helper in sync if `TCValue` changes.

### Special cases worth knowing

- **JDK8u20**: the pwntester JDK8u20 gadget is not a valid serialization stream. `FromJDK8u20Bytes` / `ToJDK8u20Bytes` (`serz/parser.go`) patch a single `TC_ENDBLOCKDATA` (`0x78`) byte after the sequence `00 7e 00 09` on the way in and strip it on the way out. The CLI exposes this via `--jdk8u20`.
- **UTF-8 overlong encoding** (`serz/tc_utf.go`): `TCUtf.OverlongSize` lets you re-emit strings as 2- or 3-byte overlong sequences while still round-tripping when read. `fromOverlongEncoding` is intentionally lenient — it falls through to the original byte if the multi-byte pattern doesn't match.
- **`TC_RESET`** (`0x79`) resets the handler table on read. The `TCContent` dispatcher treats it as a first-class content; do not coalesce it with adjacent records.

### `rmi` — JRMP (Java RMI) wire-protocol parser


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phith0n/zkar](https://github.com/phith0n/zkar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

---
trigger: always_on
description: Go library for parsing, authenticating, anonymizing, and marshalling EU Digital
---

# Agent Instructions

Go library for parsing, authenticating, anonymizing, and marshalling EU Digital
Tachograph binary files (.DDD/.V1B) into protobuf messages.

## Build

```bash
go mod tidy                # dependencies
./tools/mage test          # tests
./tools/mage lint          # golangci-lint (zero tolerance)
./tools/mage generate      # proto codegen + go generate
./tools/mage build         # full CI: download → generate → lint → test → tidy → cli → diff
```

## Skills

- **Tachograph** — `.agents/skills/tachograph/SKILL.md`
- **ASN.1** — `.agents/skills/asn1/SKILL.md`
- **Protobuf** — `.agents/skills/protobuf/SKILL.md`
- **Mage** — `.agents/skills/way-magefile/SKILL.md`

## Processing Pipeline

Five-phase pipeline with two intermediate representations:

```
Binary (.DDD)
  │  Unmarshal (binary → proto)
  ▼
RawFile (TLV/TV records with raw_data)
  │  Authenticate (signature verification at raw layer)
  │  Parse (raw records → semantic structures)
  ▼
File (semantic proto messages)
  │  Anonymize (PII removal)
  │  Unparse (semantic → raw records)
  ▼
RawFile
  │  Marshal (proto → binary)
  ▼
Binary (.DDD)
```

- **Unmarshal/Marshal** — binary ↔ `RawFile` (preserves raw bytes, no semantic
  interpretation)
- **Parse/Unparse** — `RawFile` ↔ `File` (semantic interpretation, generation
  dispatch)
- **Authenticate** — operates on `RawFile` (before parse), populates
  `Authentication` fields
- **Anonymize** — operates on `File` (after parse), replaces PII with
  deterministic test values

## Package Layout

```
tachograph.go          # public API (Unmarshal, Parse, Authenticate, Marshal, etc.)
internal/
  dd/                  # shared Data Dictionary types (TimeReal, HolderName, etc.)
  card/                # driver card TLV parsing/marshalling
  vu/                  # vehicle unit TV parsing/marshalling
  security/            # RSA/ECC certificate and signature verification
  cert/                # certificate resolution (embedded + remote)
  brainpool/           # Brainpool elliptic curves (Gen2)
  hexdump/             # hexdump ↔ binary conversion (test fixtures)
proto/                 # protobuf schemas + generated Go code
cmd/tachograph/        # CLI (cobra + charmbracelet/fang)
tools/                 # magefile, golangci-lint, buf, fetch-certs
```

## Architecture Patterns

### Options-on-Struct

All operations are methods on options structs (mirrors `protojson.MarshalOptions`):

```go
type ParseOptions struct {
    PreserveRawData bool
}
func (o ParseOptions) Parse(rawFile *tachographv1.RawFile) (*tachographv1.File, error)
```

Top-level convenience functions delegate with defaults:

```go
func Parse(rawFile *tachographv1.RawFile) (*tachographv1.File, error) {
    return ParseOptions{PreserveRawData: true}.Parse(rawFile)
}
```

### Options Embedding for Capability Inheritance

Internal options embed parent-layer options:

```go
// card.UnmarshalOptions embeds dd.UnmarshalOptions → inherits all dd unmarshal methods
type UnmarshalOptions struct {
    dd.UnmarshalOptions
    Strict bool
}
```

### Binary Parsing: Two Strategies

- **Card files (TLV)** — `bufio.Scanner` with custom split function. 5-byte
  header: 3-byte file ID + 2-byte big-endian length.
- **VU files (TV)** — direct offset slicing. 2-byte tag, then type-specific
  size calculation via `sizeOfTransferValue()`.

### Raw Data Painting (Marshalling)

When `raw_data` is present, marshal uses it as a canvas:

1. Copy `raw_data` into a fresh byte slice
2. Paint semantic fields at their known offsets
3. Unknown/reserved/padding bits preserved from original

This ensures byte-for-byte round-trip fidelity even when the binary format
contains bits not captured in the proto schema.

### Fixed-Size Layout Constants

```go
const (
    idxIssuingMemberState   = 0
    lenIssuingMemberState   = 1
    idxDriverIdentification = 1
    lenDriverIdentification = 14
)
```

Always validate: `len(data) == lenExpectedType`.

## Naming Conventions

### Functions

| Prefix       | Direction                     | Example                   |
| ------------ | ----------------------------- | ------------------------- |
| `Unmarshal*` | binary → proto (no semantics) | `UnmarshalRawCardFile`    |
| `Parse*`     | raw proto → semantic proto    | `ParseRawDriverCardFile`  |
| `Unparse*`   | semantic → raw proto          | `UnparseDriverCardFile`   |
| `Marshal*`   | proto → binary                | `MarshalDriverCardFile`   |
| `Anonymize*` | semantic → anonymized copy    | `AnonymizeDriverCardFile` |

### Constants

- `len*` — byte size (`lenHolderName = 72`)
- `idx*` — byte offset (`idxIssuingMemberState = 0`)

### Types

- `Raw*File` — intermediate TLV/TV records with `raw_data`
- `*File` — semantic structures
- `*Options` — configuration structs

## Proto Schema Conventions

See `proto/AGENTS.md` for full protobuf design guidelines. Key points:

- **Opaque API** — proto uses `API_OPAQUE` (getters/setters, not public fields)
- **AEP alignment** — follows https://aep.dev
- **Custom options** — `protocol_enum_value` maps binary protocol values to
  proto enum values for lossless round-trips
- **No unsigned integers** — use `int32`/`int64` for cross-language compat
- **`bytes` for OCTET STRING** — even single-byte values

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [way-platform/tachograph-go](https://github.com/way-platform/tachograph-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->

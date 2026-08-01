---
trigger: always_on
description: `tansu-sans-io` is the Kafka wire protocol implementation at the heart of Tansu. As its name implies, it performs **no I/O** — it operates purely on bytes, converting between raw Kafka protocol frames and typed Rust structures. This makes it suitable for embedding in any async runtime or network stack (Tansu uses `rama`).
---

# Research: tansu-sans-io Crate

## Overview

`tansu-sans-io` is the Kafka wire protocol implementation at the heart of Tansu. As its name implies, it performs **no I/O** — it operates purely on bytes, converting between raw Kafka protocol frames and typed Rust structures. This makes it suitable for embedding in any async runtime or network stack (Tansu uses `rama`).

The crate's description from `Cargo.toml`: *"A Kafka protocol implementation using serde"*.

The key insight of the design is that the entire Kafka binary protocol — serialization, deserialization, versioning, tagged fields — is expressed through serde's data model. A build-time code generator reads Apache Kafka's official JSON message descriptors and produces Rust types that `#[derive(Serialize, Deserialize)]`. Custom serde `Serializer` (`Encoder`) and `Deserializer` (`Decoder`) implementations then map between serde's abstract data model and Kafka's binary wire format.

## Source Layout

```
tansu-sans-io/
├── build.rs                 # Code generator (1,276 lines)
├── Cargo.toml
├── message/                 # 185 official Kafka JSON descriptors
│   ├── ProduceRequest.json
│   ├── ProduceResponse.json
│   ├── FetchRequest.json
│   └── ... (185 files total)
├── src/
│   ├── lib.rs               # Frame, Header, Body, Error, traits (2,128 lines)
│   ├── ser.rs               # Encoder: serde Serializer → Kafka bytes (820 lines)
│   ├── de.rs                # Decoder: Kafka bytes → serde Deserializer (1,448 lines)
│   ├── primitive.rs         # ByteSize trait
│   ├── primitive/
│   │   ├── varint.rs        # VarInt, LongVarInt, UnsignedVarInt (zigzag encoding)
│   │   ├── tagged.rs        # TagBuffer, TagField (flexible version support)
│   │   ├── tagged/ser.rs    # Tagged field serializer
│   │   ├── tagged/de.rs     # Tagged field deserializer
│   │   └── uuid.rs          # UUID (128-bit) type
│   ├── record.rs            # Record module facade + Record struct + Builder
│   └── record/
│       ├── codec.rs         # Octets, Sequence, VarIntSequence codecs
│       ├── header.rs        # Record Header (key/value byte pairs)
│       ├── deflated.rs      # Compressed record batches (wire format)
│       └── inflated.rs      # Decompressed record batches (in-memory)
├── tests/                   # 18 test files
│   ├── encode.rs            # Encoding tests
│   ├── decode.rs            # Decoding tests
│   ├── codec.rs             # Round-trip tests
│   ├── api.rs               # API-level tests
│   ├── snappy.rs            # Snappy compression tests
│   └── ...
└── benches/                 # Criterion benchmarks
```

## Build-Time Code Generation (`build.rs`)

### Input: Kafka JSON Message Descriptors

The `message/` directory contains 185 JSON files taken directly from the Apache Kafka source tree. Each file describes one API message (request or response). Example structure (`CreateTopicsRequest.json`):

```json
{
  "apiKey": 19,
  "type": "request",
  "listeners": ["zkBroker", "broker", "controller"],
  "name": "CreateTopicsRequest",
  "validVersions": "0-7",
  "deprecatedVersions": "0-1",
  "flexibleVersions": "5+",
  "fields": [
    {
      "name": "Topics",
      "type": "[]CreatableTopic",
      "versions": "0+",
      "about": "The topics to create.",
      "fields": [
        { "name": "Name", "type": "string", "versions": "0+", ... },
        { "name": "NumPartitions", "type": "int32", "versions": "0+", ... },
        ...
      ]
    },
    { "name": "TimeoutMs", "type": "int32", "versions": "0+", ... },
    { "name": "ValidateOnly", "type": "bool", "versions": "1+", ... }
  ]
}
```

Key descriptor properties:
- **`apiKey`**: 16-bit integer identifying the API (e.g., 0=Produce, 1=Fetch, 19=CreateTopics)
- **`type`**: `"request"` or `"response"`
- **`listeners`**: Which broker types handle this message (the generator filters for `"broker"`)
- **`validVersions`**: Range of supported protocol versions (e.g., `"0-7"`)
- **`flexibleVersions`**: Versions that support tagged fields (e.g., `"5+"`)
- **`fields`**: Array of field definitions, each with its own version range, type, and optional sub-fields

### Generation Pipeline

The `build.rs` (1,276 lines) uses `proc_macro2`, `quote`, and `syn` to generate Rust source at compile time:

1. **Read**: `read_value()` strips `//` comments from JSON files and parses them via `serde_json`
2. **Parse**: Converts JSON values into `tansu_model::Message` structs (which model version ranges, field metadata, nested structures)
3. **Filter**: Only processes messages where `listeners` includes `"broker"` and an `apiKey` is defined
4. **Generate**: For each message, produces:

#### Generated Artifacts

**A. The `Body` enum** — A single enum with one variant per message type:
```rust
#[non_exhaustive]
pub enum Body {
    ProduceRequest(ProduceRequest),
    ProduceResponse(ProduceResponse),
    FetchRequest(FetchRequest),
    // ... ~90 variants
}
```
Plus `From<MessageType> for Body` and `TryFrom<Body> for MessageType` impls for each variant.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tansu-io/tansu](https://github.com/tansu-io/tansu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

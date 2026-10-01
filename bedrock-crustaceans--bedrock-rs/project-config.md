---
trigger: always_on
description: What the crates are: @README.md
---

# bedrock-rs agent notes

What the crates are: @README.md
Setup, checks, commits, PRs: @CONTRIBUTING.md

The two `@` files are loaded with this one; nothing here repeats them. This file is the working
method: how to drive a change through any crate test-first, what each crate's oracle is, and the
facts no config confesses.

## The loop

Every change is one pass of red, green, refactor. The test is written before the code it tests,
and it goes **red** for the reason you are about to fix. A test that cannot fail proves nothing;
a compile error is not red.

1. **Locate the owner.** Each crate section below says where its logic lives and what data it is
   measured against. Done when you can name the file and the item you will change.
2. **Write the red test** in a `#[cfg(test)] mod tests` at the bottom of that file (or in the
   crate's `tests/` for anything that needs a fixture on disk), named for the behaviour:
   `manifest_without_dependencies_imports`, not `test_manifest`. Run only it:
   `cargo test -p <crate> <test_name>`. Done when it fails and the assertion message names the
   missing behaviour.
3. **Go green** with the smallest change that passes. The neighbour you noticed gets its own
   test and its own commit.
4. **Refactor** with the test guarding: names, extracted functions, a derive instead of a hand
   impl. Rerun the one test after each step.
5. **Widen**: the crate's suite, then the checks in `CONTRIBUTING.md`. Done when fmt, clippy and
   the tests are clean, with any excluded crate named in your report.
6. **Commit** the one change, test included.

Code with no test is not a licence to skip step 2. When the behaviour you inherit is unclear,
pin what it does today with a **characterization test**, then change it.

## Test shapes

**Real data is the oracle.** Every crate here mirrors a format Mojang defines, so the strongest
test feeds in something the real game produced and checks the crate reads it and writes it back
unchanged. Prefer, in order:

1. **Round trip a capture**: decode real bytes or JSON, encode, assert identical output, then
   assert the fields you care about. Store short captures inline; larger ones under the crate's
   `tests/`.
2. **Golden value**: build the Rust value by hand and assert it equals the parsed capture
   (`PartialEq` on the type; derive it on anything new).
3. **Error variant**: for bad input, `matches!` on the variant, never on `Display` text.
4. **Two implementations agree**: where a fast path exists beside a plain one (SIMD unpacking in
   `level`), the plain one is the oracle for the fast one over a spread of inputs.

Fixtures are named for what they build (`open_test_db()`, `vanilla_manifest()`), not `setup()`.
Nothing in a test touches the network.

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use bedrock_protocol_core::ProtoCodec;
    use std::io::Cursor;

    /// Decode, encode, and require the encoding to match the capture byte for byte.
    fn round_trip(bytes: &[u8]) -> RecordStartedPacket<crate::V2193> {
        let packet = RecordStartedPacket::<crate::V2193>::deserialize(&mut Cursor::new(bytes)).unwrap();
        let mut encoded = Vec::new();
        packet.serialize(&mut encoded).unwrap();
        assert_eq!(encoded, bytes, "encoding differs from the capture");
        packet
    }

    #[test]
    fn record_started_decodes_a_capture() {
        let packet = round_trip(&[/* bytes from a client */]);
        assert_eq!(packet.server_sound_handle, 7);
    }
}
```

The same shape serves JSON (`facet_json::from_str` then `to_string` in `form`) and LevelDB
values (`from_disk` then `to_disk` in `level`).

## Per crate

**`protocol_core` and `macros`**: the `ProtoCodec` traits, primitive codecs in
`protocol_core/src/types/`, and the derive. Red test: a tiny struct using the new piece,
asserting exact bytes both ways. A change here rebuilds every packet; check `bedrock_protocol`
after.

**`protocol`**: one hand-written file per packet, type or enum, in the newest `version/v<N>/`
that changed it; `def/versions.def.rs` says which. Oracle: bytes a real client sent. Get them
from a bug report, a proxy, or the example server, where `Connection::recv_raw` hands you a batch
before decoding. Versions are diffs: `+` added, `%` replaced, `-` removed, `^` generic over `V`;
unlisted items carry forward. After editing `def/` or `version/`, `cargo xtask && cargo fmt --all`.
The default feature is the newest version alone, so also check with `--all-features`.

**`network`**: `codec.rs` (batching), `compression.rs` and `encryption.rs` are pure functions
over `Vec<u8>` and need no socket; test them there. `encode_packets` and `decode_packets` take
compression and encryption as options, so a test can cover one layer at a time. The transport is
`raknet-tokio`, a git dependency; `examples/server.rs` is the end-to-end check for it, and for a
login flow.

**`auth`**: `AuthData::validate(oidc)` with `None` verifies nothing and only decodes claims;
with `Some` it checks the JWT against Microsoft's JWKS. `AuthOIDC::fetch` is the only network
call. Tests use a token signed by a key you generate in the test and an `AuthOIDC` built by hand
around that key; fetching stays out of tests. The `async` feature duplicates `fetch`; a change
to one is a change to both.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bedrock-crustaceans/bedrock-rs](https://github.com/bedrock-crustaceans/bedrock-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

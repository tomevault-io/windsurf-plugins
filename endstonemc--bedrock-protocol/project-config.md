---
trigger: always_on
description: Each aspect of a type has one authority. Walk them per aspect, not as a ranking,
---

# CLAUDE.md

## Sources of truth

Each aspect of a type has one authority. Walk them per aspect, not as a ranking,
and confirm against a second source before acting:

| aspect | source |
| --- | --- |
| field/class name, C++ type | **bedrock-headers** (`~/bedrock-headers`) — authoritative, no TODO needed. Branched per release (`android/rNN_uN`); a type's shape can differ from head, so read the era's branch. BossEvent's enums were `: int` through r26_u2, `: uint8_t` at r26_u3 |
| wire type, order, prefix | **EndstoneMC/protocol-docs**, on the release branch (`r26_u3`) |
| golden bytes | **gophertunnel** (see Tests) |
| dating, cross-validation | **Nukkit-MOT**, **CloudburstMC/Protocol** |

Never take a *name* from gophertunnel or CloudburstMC — they date and shape
symbols, they do not name them. Mojang/bedrock-protocol-docs is a last resort and
always earns a `# TODO: confirm against BDS`. **Do not trust `prototype`**: it has real
bugs (its StartGame `game_type` is `uvarint32` where the wire is `varint32`).

**A ref is only as good as its consumer's coverage.** CloudburstMC/Protocol is
exercised by Geyser, gophertunnel by Dragonfly — a packet neither runs has an
effectively untested codec. Agreement is evidence only where the consumer actually
exercises the packet; silence is never evidence. Check whether a codec and a test
exist before leaning on a ref. gophertunnel's `MemoryCategory` list is the standing
proof: it carries a `VR` entry BDS does not, so every constant after `Textures` is
off by one and nothing caught it.

**Read `origin/<branch>`, never the local one.** protocol-docs' local branches go
stale, and a stale dump is not merely old — it is *wrong in the direction that
looks like a real finding*. A sweep that read the working tree's `r26_u3` (four
commits behind) concluded BossEvent and MobEquipment narrowed `uvarint32` to
`uint8` between 1001 and 2168; `origin/r26_u3` says `uint8` at both, and the
whole "disagreement" was the stale checkout. Same for bedrock-headers eras: use
`git show origin/android/r26_u3:<path>`, and never `git checkout` in either repo.

**BDS is the faithful source; a community codec is not, ever.** Where a ref
disagrees with BDS, the ref is wrong until proven otherwise, and it has been
wrong repeatedly: gophertunnel writes the unlocking requirement for chemistry
recipes (BDS does not), calls `mNumIngredients` "TimesCrafted", and carries the
phantom `MemoryCategory` VR entry; CloudburstMC reads a second plain string for
`RedactableString` where BDS has `optional<string>`. Agreement between two refs
is not evidence — they copy each other.

**A ref's *read* side is evidence where its write side is not.** Codecs copy each
other's writes, but a **tolerance** records what a real server actually sent: nobody
makes a read case-insensitive, or accepts a field the spec says is absent, for bytes
that never arrive. gophertunnel's `StringConst` compares with `strings.EqualFold`,
and that single line is what proves BDS emits those constants at all — and emits them
folded, not in the C++ casing gophertunnel itself writes. Read `Reader`, not just
`Writer`, and treat a defensive read as a live-traffic observation.


**When the header and the dump disagree, read the binary.** That is the only
arbiter, and it is cheap: the IDA databases under `~/bedrock-symbols` are already
built. Two methods that worked. Decompile the writer directly, which settled the
chemistry recipes (`serialize<ShapedChemistryRecipe>::write` ends at Assume
Symmetry). Or, where symbols are absent, count code owners of a cereal field-name
string literal — `"Radius X"` and `"Radius Z"` are registered back to back in the
shape cerealizer, proving `CylinderDataPayload`'s single `Vec2 mRadii` is bound
twice and two Vec2s reach the wire.

**But a `bind` function says what is *bound*, never what is written.** Reading
`cerealizer<T>::bind` and finding a member absent from it does not make that member
absent from the wire — the serializer walks the bound set, and a cereal bug can walk
more of it than the schema intends. `cerealizer<TextPacketPayload>::bind` binds four
members and a run of `BasicFactory<TextPacketType>::bindConstInternal` entries; at 898
those const entries were **serialized**, putting twelve enumerator names on the wire
(74 of an 85-byte raw text packet) until 924 fixed it. Concluding "bound as a const,
therefore not on the wire" contradicted two codecs, the dump and a case-insensitive
read, and was wrong. To settle presence, decompile `write` and count what it emits.


**Dating shortcuts:** Nukkit-MOT annotates members `@since vNNN`. A CloudburstMC
`Serializer_v1001` that extends `Serializer_v975` and appends a field dates that
field to 1001.

**A protocol-docs branch is a network version**, not a release ordering — check its
README, and check it again after every fetch, because a branch bumps within itself:
`r26_u4` was 2168 through 1.26.44.3 and is **2169** from 1.26.45.1 (`r26_u5` = 2193,
`r26_u6` = 2207). A packet missing from the dump is not cerealised at that version,
so the dump cannot describe it at all.

## Settled — do not re-open without new evidence

Each of these was argued from the binary or the headers and closed. Re-deriving

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EndstoneMC/bedrock-protocol](https://github.com/EndstoneMC/bedrock-protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

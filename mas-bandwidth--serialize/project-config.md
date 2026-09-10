---
trigger: always_on
description: validates at runtime in release and drops invalid data, because asserts
---

<!-- HOT:BEGIN -->
## HOT — read before reasoning about this repo

WHAT: the C++ bitpacking serialization library. NOT serialize.rs / serialize.go (the ports),
NOT serialize.modern (the C++23 rewrite with compile-time schemas).

DECISIONS THAT READ AS BUGS (they are not — do not "fix" them)
- **The serialize macros hide `return false` on purpose.** Invalid data must abort the
  whole serialize function immediately, never continuing deeper or into a loop bounded by
  attacker-controlled data. The library is low-level C-style and deliberately does not use
  exceptions. Serialize functions must be `template <typename Stream>` returning bool. Do
  not propose exceptions or error codes; the early-return macro IS the mechanism.
- **~80 implicit-narrowing warnings are a deliberate style**, not neglect.
- **Asserts on the write path, runtime validation on the read path.** Write-side misuse is
  a programmer error (size buffers conservatively or pre-measure with `MeasureStream`); the
  READ path validates in release and drops invalid data, because asserts are not enough at
  a trust boundary. Do not propose hardened/checked write modes.
- **The buffer contracts are load-bearing** (owner-approved July 2026): write buffers must
  be a multiple of 8 bytes (the writer flushes qwords; bytes past the data are written only
  as zeros), and read allocations must extend at least 8 bytes past the packet (the reader
  loads 64-bit windows at byte granularity; bytes past the end are loaded but never
  interpreted). These contracts are what make the qword-flush writer and the branchless
  reader possible. Do not remove them or add tail handling to avoid them.
- `serialize_int_relative` requires strictly increasing values over the domain 0 to 2^31 - 1,
  for `previous` as well as `current`, and there are no wrap semantics. A `previous` outside the
  domain is caller error (debug asserted); a `current` off the wire outside it is refused, in
  every tier, reconstructed in a width that cannot wrap.
- **A refused read leaves a scalar destination unwritten, and fails the stream for good.** The
  first refusal poisons `BitReader`'s position past the end, so the past-end check every read
  already performs refuses every later read — the latch costs the read path nothing. `Initialize`
  clears it. Refusals decided outside the stream route through `serialize::serialize_fail`.
- `wstring` is 32 bits per character on the wire, for portability across 2/4-byte platforms.
- **The `__restrict`-qualified `this` on BitWriter::WriteBits/WriteBytes/FlushBits is a
  measured optimization (writes up to +152% in generated code), not decoration** — and the
  spelling matters: LLVM DROPS restrict on data members (a member `__restrict` is a silent
  no-op, byte-identical objects), only restrict-qualified member functions work. The
  contract (buffer must not overlap the writer object) is documented and debug-asserted;
  sanitizers cannot catch restrict violations. Upstream clang forbids a `__restrict` member
  function calling non-restrict members — spell asserts directly instead of calling
  helpers there. Do not remove the qualifiers, and do not trust member-restrict placements.

THE WRITE/READ RULE — this library is the clearest statement of it, IN ITS OWN DOCS
Glenn, 2026-07-26: "intention is on write, user is responsible to not crash or do undefined
behavior. asserts are there to help. callers responsibility. on read, obviously, we must
check." Plus Postel: "be conservative in what you send, permissive in what you receive."
serialize.h says it directly at :1477, :1525 and :1540 -- "All checking is performed by debug
asserts on write." That is the CONTRACT. I audited this header, quoted that exact line in my
notes, and still filed the write path as a defect. Do not repeat that.
DELIBERATELY ASSERT-ONLY ON WRITE, do NOT "fix":
  - BitWriter::WriteBits (:1034) and WriteBytes (:1091) -- the asserts are the whole bound.
  - BitWriter::Initialize / ctor (:994, :1013): serialize_assert( ( bytes % 8 ) == 0 ) is the
    ENTIRE enforcement of the qword-store contract that FlushBits (:1137) relies on. Hand a
    WriteStream a 100-byte buffer, write exactly 800 bits -- within capacity, violating no
    assert even in debug -- and the flush memcpys 8 bytes at offset 96, four PAST the end.
    Proved with a canary, 2026-07-26. Still the caller's responsibility: pass a multiple of 8.
    (yojimbo satisfies it on purpose at yojimbo_connection.cpp:248, `maxPacketBytes &= ~7`.)
  - serialize_copy_string / serialize_copy_wstring (:3172, :3186) with dest_size 0 -- the
    same size_t underflow as reliable_copy_string.
NOTE FOR ANY SANITIZER WORK HERE: ASan does NOT report that FlushBits overflow. It is a
partial-granule write (8 bytes at offset 96 of a 100-byte allocation) and ASan is blind to
it -- verified with a control, an identical raw memcpy is also unreported. "No ASan report"
is NOT evidence of safety in this header. Use a canary region.
THE READ PATH IS CLEAN and both independent audits agree: every BitReader assert has a real
ReadStream companion -- WouldReadPastEnd at :1666, :1688, :1720, :1755; ReadBytes bounds at
:1736 and :1741; values off the wire range-checked twice (:1669 and the serialize_int macro

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mas-bandwidth/serialize](https://github.com/mas-bandwidth/serialize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

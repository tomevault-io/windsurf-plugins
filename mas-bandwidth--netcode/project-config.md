---
trigger: always_on
description: WHAT: the C reference implementation of the netcode protocol (encrypted, connection-oriented
---

<!-- HOT:BEGIN -->
## HOT — read before reasoning about this repo

WHAT: the C reference implementation of the netcode protocol (encrypted, connection-oriented
UDP with connect tokens). NOT netcode.rs / netcode.go (the ports), NOT the crates.io crate
`netcode` (unrelated, taken 2017 — ours is `netcode-official`).

**THE PROTOCOL IS THE PRODUCT.** `STANDARD.md` is the spec every implementation follows. A
change here that alters wire behaviour breaks every port and every third-party
implementation. Spec change first, code second, ports after — never the reverse.

DECISIONS THAT READ AS BUGS (they are not — do not "fix" them)
- **One ~9,300-line file.** A deliberate style choice trading contribution ergonomics for
  trivial integration (drop in two files, no build system).
- **Flat linear scans, not hash maps**, for per-client address lookup and the encryption
  mapping search. A hash WAS considered and rejected: netcode targets ~100 players, and an
  attacker controls the keys (source addresses) and could drive a hash into its worst case.
  Linear is the hardened choice here, not the lazy one.
- **Connect-token single-use tracking is constant-time worst-case on purpose**
  (netcode.c:3701, says so in a comment). Timing must not leak whether a token was seen.
- **Per-packet socket errors are ignored deliberately.** UDP is unreliable, so a send error
  is semantically identical to a dropped packet; a persistently dead socket surfaces as a
  connection timeout through the state machine.
- **The running server has no state machine of its own** — only stopped/started
  (`netcode_server_running`). All other state is per-client. That is not an omission.
- **Global packet sequence starts at `1ULL << 63`** (netcode.c:3940) and is re-seeded on
  BOTH start and stop. Nonce-space separation: pre-connection and per-client packets share
  a key, so their nonces must not collide. Seeding on create only was the AEAD nonce-reuse
  bug fixed in 1.4.0. Keep every seeding site.
- **Replay protection advances the window only AFTER authentication** (netcode.c:1863,
  1907). The cheap pre-decrypt reject is an optimisation; moving the window advance before
  auth would let spoofed plaintext sequence numbers poison it.

SECURITY: netcode 1.3.5 and earlier carry the nonce-reuse issue above; see SECURITY.md for
affected versions and which channels still serve them.

KNOWN OPEN DEFECTS -- do not rediscover these, and do not assert them in tests
- #173 `netcode_generate_connect_token` bounds `num_server_addresses` with `netcode_assert`
  only. Under `-DNDEBUG` (the release build, and what Debian ships) the asserts vanish and
  the parse loops write past a 32-element STACK array. Note the same loop body already
  returns NETCODE_ERROR for a bad address string, so the inconsistency is within one
  function. Caller-precondition, not attacker-controlled by design -- not filed as an
  advisory.
- #174 `netcode_parse_address("[::1")` returns OK and yields `::1`. The opening bracket is
  skipped whether or not it was ever closed. The suite already rejects `[`, `[]` and `[]:`,
  so this is a gap, not deliberate leniency. Low severity.
Both were found by WRITING TESTS against a release build, not by reading the source.
Neither is asserted in mas-bandwidth/apt's autopkgtest on purpose: the shipped `-DNDEBUG`
library does not promise the first, and the second is unfixed. A packaging test that
asserts an unshipped fix makes a faithful package look broken.

THE WRITE/READ RULE — read this BEFORE reporting any assert as a missing bounds check
Glenn, 2026-07-26: "intention is on write, user is responsible to not crash or do undefined
behavior. asserts are there to help. callers responsibility. on read, obviously, we must
check." Plus Postel: "be conservative in what you send, permissive in what you receive."
  WRITE / caller-supplied -> the CALLER validates. An assert-only bound is the DESIGN, and
    -DNDEBUG removing it is correct. Do not add runtime checks here.
  READ / off the wire     -> the library checks at runtime, for SAFETY (bounds, sizes).
    Permissive about format variation; strict about never crashing.
This repo states the policy itself at netcode.c:4101-4103 -- "an out of range value here
must not get through in release builds where asserts compile out" -- attached to the entry
points that were hardened (3387, 4104, 4998, 5010, every client_index accessor). I audited
this file, quoted that comment, and STILL filed the write-path asserts as defects. Do not
repeat that.
DELIBERATELY LENIENT, do not "fix": netcode_parse_address("[::1") returns OK and yields
::1. That is permissive-on-receive and intended (#174 closed as by-design).
netcode_generate_connect_token WAS genuinely missing its runtime check -- it was the one
entry point skipped in the hardening pass above. Fixed; regression test
test_generate_connect_token_out_of_range.
GOTCHA: netcode_set_assert_function( NULL ) is a landmine. netcode_assert calls the pointer
with NO null guard (netcode.h:350-356), so NULL turns the next failing assert into a crash.
Restore &netcode_default_assert_handler instead -- which is what netcode.c:6984 already does.
A custom handler MAY return (documented netcode.h:362-366); that is the only way a test with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mas-bandwidth/netcode](https://github.com/mas-bandwidth/netcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->

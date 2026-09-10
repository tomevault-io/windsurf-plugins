---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working
with code in this repository.

How to work here — what the issue tracker takes, the prose style, and how
a pull request is opened, corrected and landed — is `CONTRIBUTING.md`,
the same file in every repository of the organization up to its last
section; that section, *This repository in particular*, is this tree's
and holds the environment, the commands and what gates a merge.
Repository configuration is `REPOSITORY.md`: read it before changing a
workflow, a branch rule or a setting; writing code does not need it.
Reviewing is `REVIEWING.md`, and `/review` is that file as a command;
read it before reviewing a pull request and before opening one, since it
is what the pull request will be answered against.

## Architecture

Pure-Python bitcoin cryptography, with secp256k1 arithmetic delegated to
the `btclib_secp256k1` cffi bindings — and delegated conditionally,
which is the single most important thing to know before touching
`src/btclib/curves/` or `src/btclib/ecc/`:

- `curves.curve.mult`, `double_mult_var` and `multi_mult_var` call the bindings
  for secp256k1 and any point of it, a zero scalar and the point at
  infinity excepted: libsecp256k1 has no scalar for the one and no public
  key for the other, so those two — and a sum landing on infinity — are
  recognized before the call and answered by the Python arithmetic of
  `curves/curve_group.py`, which is what every other curve runs
- `ecc.dsa.sign` calls them for secp256k1 with sha256, lower-s, and no
  caller-imposed nonce; `ecc.ssa.sign` for secp256k1 with sha256, a
  message of **any** size, and no sign-to-contract commitment. The size
  used to be a third condition, which was issue 169 and the four
  arbitrary-size vectors BIP340 gained in 2023-04: the bindings'
  `ssa.sign` takes a 32-byte message, but `ssa.sign_custom` beside it
  takes any, and `ssa.verify` always did — it was the gate in front of
  them that sent those four down the Python path
- the Python path is not dead code and not constant-time: it serves every
  other curve, other hash functions, and caller-supplied nonces, and the
  test suite validates it *against* the bindings, which are the authority
  on the answer. `SECURITY.md` publishes this as a known limitation

Layers, roughly bottom-up: `curves/` (curve arithmetic) → `ecc/` (dsa, ssa,
bms, borromean, pedersen, rfc6979/bip340 nonces). At the key boundary,
`base58` and `bech32` are the low-level codecs; `bip32/` depends on
`base58`; `b58` and `b32` depend on `to_pub_key`, which depends on
`to_prv_key`, and neither converter depends on `bip32/` -- so importing
`btclib.b58` does not put `btclib.bip32` in `sys.modules`, which is the
measurement behind the arrow. What each spelling of a key resolves
through is the module that defines it: a WIF is Base58Check with a prefix
and a flag, so `b58.prv_key_data_from_wif` reads it and
`b58.wif_from_prv_key` writes it; an extended key is BIP32's format, so
`bip32.prv_keyinfo_from_xprv`, `bip32.pub_keyinfo_from_xpub`,
`bip32.pub_keyinfo_from_xkey` and `bip32.point_from_xpub` are the parse,
and a caller holding one calls one of them and passes the scalar or the
point on (issue #1188). The scalar and the curve point in their octet
spellings are not a converter's either: they are facts about the curve,
so `curves.scalar_from_prv_key` and `curves.point_from_pub_key` read
them, and what is left to `to_prv_key` and `to_pub_key` is the network
and the compression a record carries and a key does not. `slip132` sits
above the address encodings, beside `bip44`. `mnemonic/`, `script/`,
`tx/`, `block/`, `psbt/` and `descriptors` build on those layers.
`alias.py` holds the type aliases the public API accepts, and much of the
surface takes "anything convertible" rather than one type.

Each of those pairs is one idea split in two, and each split runs one
way only: `curves/` is arithmetic and `ecc/` is what is built on it;
`base58` and `bech32` are codecs with no bitcoin in them, `b58` and `b32`
the bitcoin semantics on top. `ecc` imports `curves`, `b58` imports
`base58`, `b32` imports `bech32`, and never the reverse. The README
carries the same layout as a table, and each of these modules states its
own direction in its docstring, wherever that direction changes.

## The primary checkout is the maintainer's

**Never work in it.** No edit, no `git add`, no commit, no branch switch,
no rebase, no `git stash`, no `pre-commit run` — the hooks fix files in
place. It is the maintainer's window on the tree: whatever is open in
their editor, whatever they have half-staged, and the branch they are
looking at are theirs, and one working tree has one index and one HEAD to
lose. Reading it is fine — `git log`, `git show`, `git diff`, `gh`, and a
`git fetch`, which writes refs and leaves the work tree alone.

So a `grep` or a `Read` against the checkout's files answers for whenever
it was last brought forward, not for now. The read that cannot go stale
is `git show origin/main:<path>`: it answers from the ref `git fetch`
just moved, never from the tree. Where the checkout has to be current
rather than merely readable, a fast-forward of a clean `main` brings it
up:

```shell

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [btclib-org/btclib](https://github.com/btclib-org/btclib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->

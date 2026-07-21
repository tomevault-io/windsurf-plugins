---
trigger: always_on
description: Guidance for coding agents working in `github.com/xssnick/tonutils-go`.
---

# AGENTS.md

Guidance for coding agents working in `github.com/xssnick/tonutils-go`.

This document is intentionally repo-specific. Follow the existing code and package boundaries here before applying generic Go advice.

## Project Shape

`tonutils-go` is split by protocol layer and domain:

- `address`: TON address parsing, formatting, flags, bit helpers.
- `tl`: TL schema registration, loader/serialization primitives.
- `tlb`: TLB loaders/serializers built on top of cells and struct tags.
- `tvm/cell`: core cell, slice, builder, dict, proof, BoC primitives.
- `tvm`, `tvm/vm`, `tvm/op/*`, `tvm/tuple`, `tvm/vmerr`: TVM emulator and opcode implementation.
- `adnl`, `adnl/dht`, `adnl/overlay`, `adnl/rldp`: transport and overlay protocols.
- `liteclient`: liteserver connection pool, balancing, sticky contexts, config loading.
- `ton`: high-level blockchain API on top of `liteclient`, proofs, block/account/transaction access.
- `ton/wallet`, `ton/nft`, `ton/jetton`, `ton/dns`: domain clients built on `ton`.
- `toncenter`: HTTP-based client as a separate access path.
- `example`: runnable user-facing examples, kept small and practical.

When adding code, keep it at the lowest layer that owns the responsibility. Do not pull high-level TON client concerns into `tl`, `tlb`, `tvm/cell`, or transport packages.


## API design

- Prefer linear APIs.
- Do not use tri-state returns like `(value, ok, err)` in storage and domain code.

- When data may be absent, use `(value, error)` and return a dedicated not-found error such as `ErrNotFound`.

- If `err == nil`, the returned value must already be valid and ready to use.
- Do not design APIs where `err == nil` but the caller still has to inspect `ok` or check the value for `nil`.

- Keep boolean returns only when they represent a real property or business flag, not presence or absence of data.

- Do not add wrapper APIs, aliases, or renames that do not simplify the code.
- Avoid patterns like:
  - `type X = Y`
  - `type Options = ImplOptions`
  - `Open -> OpenImpl`
- Name public types and functions correctly once and use them directly.

- Do not introduce internal conversion helpers like `fromX` and `toX` unless there is a real format boundary, protocol boundary, or external API boundary.
- Inside the project, prefer using the actual types directly.

## Code style

- Add empty lines between logical blocks inside functions.

- Do not make excessive nil checks.
- Check only what can really be nil logically. Do not check obvious input values for nil, which are intended to have a value.
- Do not check obvious value inputs for nil.
- Do not add defensive nil/self-guard checks for required inputs, even when they are pointers. If a pointer is part of an internal invariant and the caller must provide it, use it directly.
- Required fields should not get fallback encoding or "just in case" nil handling.
- Treat low-level code as potentially hot. Add runtime checks only when they protect a real boundary, enforce an invariant that can be violated in normal use, or prevent a meaningful safety issue. Avoid defensive checks that duplicate earlier validation or guard states that cannot happen through the package API.

- Keep code simple and optimized.
- Add abstractions only when they remove real duplication or represent a real boundary.

- Prefer straightforward Go-style code.
- Keep the control flow linear and easy to read.
- Avoid unnecessary indirection.
- 
- Beware of fallbacks, each fallback must have a strong reason why it is necessary and commented.
- Do not leave useless short wrapper methods, less calls are better unless it is big piece of code which is used in many places
- Do not handle both `T` and `*T` in type switches "just in case". Pick the concrete representation owned by that layer, convert once at a real boundary, and pass that concrete type through the rest of the code. Existing protocol-boundary compatibility helpers are exceptions, not patterns to copy.

## Protocol types

- Any type used in `tl.Register(...)` must be public and named with a capital letter.

## Tests

- Keep test-only constants, hooks, and helpers in `_test.go` files.
- Do not leave test-only code in normal production files.

## Architecture Rules

- Treat `tvm/cell`, `tl`, and `tlb` as foundational packages. They should stay reusable and mostly independent from higher-level TON client logic.
- Keep transport and business logic separate. `liteclient` handles connectivity, node selection, stickiness, retries/timeouts at transport level; `ton` handles proof-aware blockchain semantics on top.
- Put domain-specific smart-contract clients under `ton/*` packages, not in `liteclient` or low-level protocol packages.
- Keep compatibility wrappers when API evolution is cheap. This repo already preserves deprecated aliases, wrapper methods, and compatibility files instead of breaking callers aggressively.
- File naming usually mirrors protocol version or variant: `client-v2.go`, `client-v3.go`, `v5r1.go`, `overlay-adnl.go`, `item-editable.go`. Follow that pattern instead of inventing abstract names.

## Coding Style Seen In This Repo

- Use idiomatic Go with `gofmt`. Do not mass-reformat unrelated files just to normalize import groups or spacing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xssnick/tonutils-go](https://github.com/xssnick/tonutils-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->

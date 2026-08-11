---
trigger: always_on
description: ├── common/           # delta-core: shared state types, crypto, serialization
---

# Delta - Agent Guide

## Repository Structure

```
delta/
├── common/           # delta-core: shared state types, crypto, serialization
├── contracts/
│   └── site-contract/  # Freenet contract: validates state, handles updates
├── delegates/
│   └── site-delegate/  # Local agent: stores signing keys, signs pages
├── ui/               # Dioxus web UI (compiled to WASM)
├── published-contract/ # Committed web container WASM + params
├── legacy_delegates.toml  # Migration entries for delegate WASM changes
├── legacy_contracts.toml  # Migration entries for contract WASM changes
├── scripts/          # add-migration.sh, add-contract-migration.sh, sync-wasm.sh
└── Makefile.toml     # Build tasks
```

## Key Concepts

### Site Identity

A site is identified by a 10-character **prefix** derived from the owner's Ed25519 public key: `base58(pubkey)[..10]`. This prefix IS the contract parameters. The full contract key is `BLAKE3(BLAKE3(site_contract.wasm) || CBOR({prefix}))`.

Anyone who knows the prefix can compute the contract key because the WASM is public.

### State Design

```
SiteState {
    owner: VerifyingKey,           # Owner's public key
    config: SignedConfig,          # Site name/description (signed)
    pages: BTreeMap<PageId, Page>, # All pages
    next_page_id: PageId,          # Monotonic counter
    deleted_pages: BTreeMap<PageId, SignedPageDeletion>,  # Tombstones
}
```

All content is signed by the owner. Pages have stable IDs that don't change on rename.

### CRITICAL: All State Fields Must Be Authenticated

**Every field in the contract state MUST be covered by a cryptographic signature.** Freenet contracts run on untrusted peers who can modify state. The contract validates signatures, but only for fields included in the signing bytes. An unsigned field is world-writable.

When adding a new field to any signed struct (Page, SignedConfig, SignedPageDeletion):
1. Include it in the signing bytes immediately
2. If backwards compatibility is needed, use a versioned signature (try v2 first, fall back to v1)
3. Add a test verifying the new field is covered by the signature

Page signatures use v2 format (`delta:page:v2:`) which covers: page_id, title, content, updated_at, order. V1 fallback (without order) exists for pre-existing pages.

**`updated_at` must be strictly greater than the page's current `updated_at`.** `apply_delta` and `merge` in `delta-core` dominate equal timestamps with `>=`, so an UPDATE whose `updated_at` matches what's already in state is silently dropped on the network. Any UI path that produces a page UPDATE (`save_current_page`, `rename_page`, `swap_page_order`, …) MUST route through `next_page_updated_at` in `ui/src/state.rs`, which computes `max(now_secs(), existing + 1)`. Calling `now_secs()` directly is a recurrence of the reorder bug Ivvor reported on 2026-04-29 (silent same-second collisions).

**Page `order` invariants (for `swap_page_order` / `create_page` in `ui/src/state.rs`):**

1. `swap_page_order` MUST sign and propagate a fresh page-UPDATE for **every** page whose order changes — not just the two pages clicked. When ANY page on the site is still at `order == 0` (legacy v1-signed pages, or pages created before the order field existed), the swap also performs a one-time site-wide migration to explicit orders `(10, 20, 30, …)` sorted by `(current_order, page_id)` to match the sidebar. Skipping the propagation step is the bug Ivvor re-reported on 2026-05-03 — the local view looked correct but unmigrated pages remained at `order = 0` on the network and clumped to the front of the sidebar after a refresh.

2. `create_page` MUST assign `order = max(existing) + ORDER_STEP` via `next_create_order` (never `0`). Issuing `0` re-poisons a migrated site and re-introduces the front-of-sidebar clumping symptom.

3. The orchestration helper `plan_swap` derives `pages_to_sign` from the diff between current and new orders, so a regression that re-narrows the sign set (e.g. back to "just the two clicked pages") is caught by the unit tests — not just by the lower-level `compute_swap_orders` tests.

**Delegate-response routing MUST use signature verification, not `CURRENT_SITE`.** `handle_signed_page` / `handle_signed_deletion` / `handle_signed_config` in `ui/src/freenet_api/delegate.rs` look up the owning site by checking the signature against every known owner's pubkey (`find_owner_for_signed_*`). Earlier code keyed `PENDING_UPDATES` by `(CURRENT_SITE, page_id)` and consumed the entry on first response; concurrent requests for the same page silently dropped subsequent UPDATEs, and a mid-flight site switch routed a signed page into the wrong site's local state. Verification-based routing handles both correctly without a delegate WASM change.

### Page Links

- `[[2]]` - renders as current page title, auto-updates on rename
- `[[2|custom text]]` - renders as "custom text", never changes
- `[[Page Title]]` - title lookup, renders as title

Autocomplete inserts `[[id]]` format.

### Delegate Storage

The delegate stores:
- **Signing keys**: `delta:signing_key:{prefix}` - per-site Ed25519 private keys (legacy: `delta:signing_key`)
- **Known sites**: `delta:known_sites` - list of sites with prefix, name, role, contract key

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freenet/delta](https://github.com/freenet/delta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->

---
trigger: always_on
description: Instructions for any coding agent working in this repository. This file is
---

# AGENTS.md — nowhere

Instructions for any coding agent working in this repository. This file is
tool-agnostic on purpose: no vendor-specific syntax, no file-reference macros, no XML
tags. It should read the same to every model and every editor.

Nested `AGENTS.md` files exist under `apps/*`, `packages/*` and `relay/`. The nearest
one to the file you are editing wins; it adds to this file rather than replacing it.

---

## 1. What this is

**ElseWeb** is a federated protocol for a social layer over the existing web, plus
its reference implementations. This repository's job is narrow: **develop the
protocol, and keep its documentation accurate.** End-user products built on top of
it — including the first one already live — are out of scope here and live in
their own repositories; they are just consumers of `packages/protocol` and
`packages/client` like anyone else's client would be.

What this repo ships:

- **The protocol** (`packages/protocol`) — the normative wire contract.
- **A reference relay** (`relay/`) — the MVP binding is a Supabase edge function,
  but the HTTP contract is written to be a **standard** anyone can implement and
  self-host. Supabase is the reference implementation, not the architecture.
- **A reference client** (`packages/client`) — everything a client does that is not
  UI: identity, relay pool, publishing, reading, ranking.
- **A generic protocol client** (`apps/extension`) — a Manifest V3 Chrome extension.
  On a site with an adapter (x.com for the MVP) it shows a "share with ElseWeb"
  control next to the host's own composer; content shared through it **never
  reaches the host site** — it is written to the configured relay set, and every
  other user of the extension sees it while on the same page. Beyond that social
  surface, the extension also holds a protocol identity that can be imported from
  any other client that generated one, and can act as a bridge — talking to a
  local OpenAI-compatible endpoint (Ollama and similar) so that WebGPU/Ollama-style
  compute providers can be reached over the same protocol. That compute-bridge
  mechanism is a roadmap direction, not yet specified at the protocol level.

Flow (social sharing): `shared content -> relay -> other clients on the same page`.

## 2. Non-negotiables

These define the product. If a change would break one of them, stop and ask before
writing code.

1. **Shared content never goes to the host site.** Not to x.com, not to any other host.
   The host site's own posting/sharing flow is never invoked, prefilled, or triggered.
2. **The host page's DOM is not polluted.** Everything injected lives inside a shadow
   root. No global CSS, no writing into the host's class/id namespace, no mutation of
   host elements beyond an anchor point needed to mount.
3. **Clients speak to a set of relays, never one.** The relay set is user-editable at
   any time, and single-relay operation is the degenerate case of a set of one — never a
   separate code path. No relay URL, key, or SDK call outside `relay/` and the configured
   transport.
4. **No participant is privileged by the protocol.** ElseWeb's own relay is a relay
   like any other. Our product decisions — an attestation-gated feed, a paid membership —
   are expressed with mechanisms available to everyone, so a community client can make
   different choices against the same network.
5. **Identity is the keypair, and it belongs to the user.** Keys are generated
   extractable so a user can carry one identity across the extension, the site and mobile.
   A private key is never sent to a server, encrypted or not.
6. **Site adapters describe behavior, not data.** A site with no adapter falls back to
   the generic adapter. The extension degrades on unknown sites; it never dies on them.

## 3. Repo map

| Path | Contains | Does not contain |
|---|---|---|
| `apps/extension` | MV3 extension: WXT entrypoints, content script, background worker, popup/options, plain Svelte UI — the generic protocol client, including identity import and the local-endpoint compute bridge | SvelteKit, site selectors, protocol schemas, relay logic |
| `packages/protocol` | `SPEC.md` and its implementation: event schemas, canonical serialization, crypto, proof-of-work, page identity | Browser APIs, network calls, storage |
| `packages/client` | Relay pool, publishing, reading and merging, key management, ranking — everything a client does that is not UI | DOM, platform storage APIs, site selectors |
| `packages/adapters` | Per-site adapters (x.com + generic fallback) | Network calls, storage, extension internals |
| `relay/` | Reference relay: a portable core in `src/`, a Supabase binding in `supabase/`, migrations and RLS policies | Client code, anything Supabase-specific inside `src/` |

`packages/client` is platform-independent (storage and clock are injected ports) so
that any host — the extension here, or an external product in its own repo — can
consume it unchanged. That is the reason client logic lives in a package rather than
in `apps/extension`.

`relay/` is a workspace member so its tests run with everyone else's. The dependency
direction is unchanged: nothing in `relay/src` may import a client, and `@elseweb-app/client`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elseweb-app/nowhere](https://github.com/elseweb-app/nowhere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->

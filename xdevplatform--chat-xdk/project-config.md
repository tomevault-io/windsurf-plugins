---
trigger: always_on
description: One Rust core (`chat-xdk-core`) plus thin bindings for **Python, JavaScript/WASM,
---

# AGENTS.md — working in chat-xdk

One Rust core (`chat-xdk-core`) plus thin bindings for **Python, JavaScript/WASM,
Go, .NET, JVM**. Three tenets govern every change: **it is a cryptography
library, not a bot SDK**; **all bindings stay at parity**; **the public API stays
small and uniform**. The rest of this file is the invariants that keep those true.

## 1. Crypto-only boundary

This SDK turns plaintext + keys into encrypted blobs + signatures, and back. It
does **not** do networking, messaging, or application logic. Per
[`crates/core/src/lib.rs`](crates/core/src/lib.rs) and
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md §1), the SDK does **not** own:

- HTTP / REST calls to `/2/chat/*` — the caller's code
- webhook servers, polling loops, retries
- message persistence, ordering, and replay dedup (the caller dedups on the
  signed `messageId`; `sequence_id`/`created_at` are unsigned backend metadata)
- key persistence
- OAuth / token management

Do not add an HTTP client, a transport, a "bot" helper, or anything that knows
about endpoints to any crate under `crates/`. That code belongs in the caller
(see `examples/`, which are demos — not part of the shipped SDK). If a task seems
to need networking inside the core, it is in the wrong layer.

## 2. Binding parity

`chat-xdk-core` is the **single source of truth**. Bindings marshal arguments
and delegate; they must **not** reimplement crypto, key handling, or protocol
logic. A change that lands in one binding but not the others is incomplete.

When you add or change SDK behavior:

1. Implement it once in `crates/core` (`ChatCore`, or `Chat` for Juicebox).
2. Surface it in **every** binding with matching behavior and signatures:
   Python `crates/pyo3`, JS/WASM `crates/wasm`, Go `crates/go` → `go/chatxdk`,
   .NET + JVM (both over the `chat_xdk_dotnet` cdylib in `crates/dotnet`,
   `crates/jvm`).
3. Use each language's idiom for naming only: `snake_case` (Rust/Python),
   `camelCase` (JS/Java), `PascalCase` methods (.NET/Go).
4. Update the per-binding tables in [`docs/API.md`](docs/API.md) in lockstep.
5. Extend the parallel per-binding test suites so the new surface is covered
   everywhere (see §5).

## 3. Clean API

Keep the public surface small and uniform. The flow every binding exposes:
key management (`import`/`export`, `setup`/`unlock`), conversation keys
(`prepareConversationKeyChange` / `decryptConversationKey`), `encryptMessage`/reply,
and two decrypt paths whose contracts are **identical across bindings**:

- `decryptEvents(events, signingKeys)` — batch (initial load / pagination).
  Self-extracts conversation keys from `KeyChange` events, **never throws**
  (per-event errors collected in the result).
- `decryptEvent(event, conversationKeys, signingKeys)` — single event with
  pre-cached keys. **Throws** on failure.

Don't add speculative configurability or one-off helpers.

## 4. Security invariants — never weaken to make something pass

- **Verification is not optional.** `decryptEvent` rejects unverified events by
  default; a below-floor key version must never verify; an invalid signature must
  never yield plaintext. Verify against caller-supplied signing keys — **never** a
  key carried inside the event. Verification covers every signed event type, not
  just messages. Signature mismatches are bugs to investigate, not checks to silence.
- **Key downgrade protection is automatic.** Conversation-key versions move
  forward only (monotonic high-water mark held for the `Chat` lifetime); the
  ordering authority is the *signed* key version, not backend sequence numbers.
  Never accept an older version over a newer one, and never add an API that can
  lower the floor.
- **Wire formats are frozen.** The bytes the SDK produces and consumes stay
  compatible with other X Chat clients and with previously stored ciphertext and
  keys. Do not change a wire format — nonce sizes, AAD, HKDF `info`, key
  derivation, secretbox/secretstream layout, the signature payload, or Juicebox
  `UserInfo` — even when the change looks more standards-correct; it breaks
  interop and decryption of stored data. A format change is valid only as a
  versioned, dual-read migration (e.g. a new `signature_version`).
- **All wire parsing is bounded and runs before any crypto.** Untrusted Thrift
  goes through the bounded reader (`crates/core/src/protocol/safe_reader.rs`):
  reject negative/oversized lengths and container counts, never panic on unknown
  union variants or field IDs. Don't route parsing around it, relax the caps, or
  replace the pinned patched `thrift` fork with upstream. New parsing paths get a
  fuzz target.
- **Known protocol limitations are documented, not re-litigated.** No forward
  secrecy / PCS, message encryption without context-binding AAD, and
  single-shared-key group conversations are intentional protocol-level
  constraints recorded in [`docs/CRYPTO.md`](docs/CRYPTO.md) (Known Limitations).
  Don't "fix" them in a binding or claim guarantees the protocol doesn't provide;
  closing them requires a versioned protocol change.
- **No forward secrecy / no post-compromise security** — by protocol design.
  Don't claim otherwise in code or docs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xdevplatform/chat-xdk](https://github.com/xdevplatform/chat-xdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->

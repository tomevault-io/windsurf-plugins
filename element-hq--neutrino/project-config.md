---
trigger: always_on
description: The project is a minimal rust-based Matrix homeserver which will be embedded into an Android device using UniFFI. The server is only capable of sending and receiving message / state events, meaning this project only implements a subset of the Matrix specification. The specification targeted is https://spec.matrix.org/v1.18/ - strictly only the Client-Server API and Server-Server API.
---

## project

The project is a minimal rust-based Matrix homeserver which will be embedded into an Android device using UniFFI. The server is only capable of sending and receiving message / state events, meaning this project only implements a subset of the Matrix specification. The specification targeted is https://spec.matrix.org/v1.18/ - strictly only the Client-Server API and Server-Server API.

The server only targets room version 12, along with MSC4242: State DAGs https://github.com/matrix-org/matrix-spec-proposals/pull/4242 . This means the Server-Server API does not need to implement /event_auth, /state or /state_ids. EDUs and End-to-End encryption are NOT implemented, but MUST be stubbed out at the HTTP handler layer to ensure the client application functions correctly. Ruma https://github.com/ruma/ruma MUST be used. The homeserver will be running in a trusted network. This means events MUST NOT have signatures and signature checks should not be run, which means servers DO NOT need a server signing key. The Client-Server API is never exposed on the network, it’s entirely embedded in the mobile device. As such, there is no need to make the Client-Server API performant or have any kind of access control. Registration and Login should be stubbed out.

see PLAN.md for current status and task breakdown.
read PLAN.md at the start of every session before doing anything else. 


## stack
- axum (routing + handlers)
- tokio (async runtime)
- serde + serde_json (serialization)
- thiserror (error types)
- uuid (id generation)
- Ruma for Matrix types
- tracing + tracing-subscriber (logging)
## crate structure - keep big dependencies separate (namely UniFFI) to improve compile times and avoid rebuilds, etc.
Crates are organised by scope, narrowest first: `event` = everything event-scoped, `room` = everything single-room-scoped, `engine` = everything multi-room-scoped. `ctl` is the orthogonal server-wide control plane. The dependency stack is a clean gradient: ctl/event (base, no internal deps) → room → engine → http → main → ffi/neutrino.

neutrino-ctl - server control plane: `Config` (+ `from_env`) and `Command` (host-pushed control). Zero dependencies; base sibling of neutrino-event.
neutrino-event - everything event-scoped: the canonical `Event` PDU, content/reference hashing + event-id derivation, event→ruma views, wire-format parsing + provider-free semantic validation (`parse_event`/`validate_pdu`), the server-side `EventBuilder` + inbound `from_wire`, and `FormatError`. No internal deps. `ROOM_VERSION_ID` and the shared `now_ms()` clock util also live here.
neutrino-room - everything single-room-scoped: v12 auth rules, state resolution (state-res), provider-backed reference validation (`validate_references`), `RoomCore` (per-room state machine), and the `StateProvider` read trait. Depends on neutrino-event.
neutrino-engine - everything multi-room-scoped: the room runtime. `RoomRegistry` + per-room actor, inbound worker + outbound sender, anti-entropy reconcile + state-DAG gapfill, and the federation transport ports (`FederationTransport`, `MissingEventsFetcher`) so the engine never names reqwest.
neutrino-store - storage trait (`StorageBackend` + the fine-grained per-area store traits, `WithStateProvider`).
neutrino-store-sqlite - SQLite implementation of the storage traits (`SqliteStore`) + the SQLite-backed `StateProvider`.
neutrino-http - top-level router + C-S and S-S handlers, could potentially be split into c2s and s2s APIs in the future (the `federation/` subtree is already a self-contained S-S unit).
neutrino-main - server entrypoint, common between neutrino and neutrino-ffi; composes the stack and re-exports `Config`/`Command` from neutrino-ctl.
neutrino-ffi - UniFFI binding layer, calls into neutrino-main and neutrino-lb
neutrino - local development binary
neutrino-lb - Low-bandwidth bidirectional proxy - translates server-to-server HTTP + JSON requests into CoAP + CBOR (CBOR could be done in HTTP layer?) - see MSC3079 https://github.com/matrix-org/matrix-spec-proposals/blob/kegan/low-bandwidth/proposals/3079-low-bandwidth-csapi.md 

## coding rules
errors
- all errors use thiserror. no anyhow.
- all handlers return Result<Json<T>, AppError>
- AppError variants: NotFound, BadRequest, Internal. map to 404, 400, 500.
- never use .unwrap() or .expect() in handler or storage code.
-	 potentially `#![deny(clippy::unwrap_used)] in handler crate to enforce this

storage
- handlers never touch store directly. always go through StorageBackend trait.
- sqlite layer implemented in neutrino-store-sqlite
- do not introduce any other database dependency without explicitly being asked to.

async
- no blocking calls inside async fns. use tokio::task::spawn if needed.
- do not add unnecessary .clone() — check if a reference works first.

style
- run cargo fmt before finishing any task
- run cargo clippy and fix all warnings before finishing any task
- no dead code. no unused imports.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [element-hq/neutrino](https://github.com/element-hq/neutrino) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->

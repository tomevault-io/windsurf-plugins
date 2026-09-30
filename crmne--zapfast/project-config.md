---
trigger: always_on
description: ZapFast is a small native WhatsApp client: Rust, egui, and the
---

# ZapFast agent guide

ZapFast is a small native WhatsApp client: Rust, egui, and the
[whatsapp-rust](https://github.com/oxidezap/whatsapp-rust) library for the
protocol. These notes are for coding agents and new contributors.

## Product boundaries

- Keep it a small native client. No browser engine, no telemetry, no
  hosted backend, no ZapFast-operated account system. Features never send
  message content to a third party.
- Do not vendor, fork, or patch upstream crates (egui, epaint, whatsapp-rust)
  in this repository. Fix them upstream.
- The protocol comes from whatsapp-rust. Do not reimplement pieces of it
  here, and do not advertise a capability merely because a protobuf field
  for it exists.
- Do not broaden a task into adjacent features or a general refactor.
  Preserve existing user behaviour unless the task changes it.

## Privacy

- The user's archive is personal data. Do not read chat rows, message
  bodies, contacts, or other user content out of `archive.db` or any
  exported log, not even read-only. Schema, column existence, and row
  counts are fine; message contents are not.
- When a bug report or feature needs the user's data, hand the user the
  query or command to run and let them report the result back.
- Never log message contents, phone numbers, keys, or QR payloads at a
  level that ships (see the definition of done); treat existing
  captures of them the same way.

## Architecture

- `src/ui/` draws views and pushes `model::Action`s; `src/app.rs` applies
  them after the frame. Never mutate application state from inside a view
  beyond the view's own fields (composer text, search text, flags).
- `src/backend.rs` is the interface's handle to a tokio runtime on its own
  thread; `src/backend/worker.rs` runs there. It owns the whatsapp-rust
  `Bot`, the message archive, downloads, and profile pictures. The two
  sides talk only through `Command` (interface to runtime) and `Event`
  (runtime to interface); every event wakes the window through `Waker`.
- `src/archive.rs` is the SQLite store of chats, messages, contacts, and
  privacy-id mappings. WhatsApp replays history once, at link time, so the
  archive is the only copy. It keeps each message's raw protobuf because
  the keys to fetch an attachment live in it. `src/archive/encryption.rs` opens
  the archive with SQLCipher and a random key stored in the OS keyring. Plaintext
  migration checkpoints the old WAL and verifies an encrypted staging file before
  atomic replacement. A locked or missing key stops linking; never fall back to
  a disposable archive. Tests use fixtures and mock credentials only.
- `src/model.rs` holds the app's own types. Views never touch a protobuf;
  the worker translates in `classify()` and `parse_conversation()`.
- Favorite chats sync with the phone through the `favorites` app-state action
  (RegularHigh), which carries the whole ordered list: `Event::FavoritesUpdate`
  replaces ours and `send_app_state_action(&schemas::FAVORITES, ..)` writes it.
  `archive/favorites.rs` keeps the list in order with each entry's JID as the
  phone named it, plus a queue of changes made here; a phone list applies
  (unless older than the newest applied) and the queue replays on top.
  `backend/worker/favorite_chats.rs` sends one list at a time with backoff and
  never before the phone's list is known: the first connection reads
  RegularHigh once as a snapshot, and its completion on the same queue as the
  replayed mutations means a phone without favorites. A blind write would
  replace the phone's list. Channels are never favorites. The Favorites chip
  follows the list order; pins stay global and first in every chip.
- Interactive messages are parsed in `backend/worker/interactive.rs`. Views receive
  labels and local capabilities, never protocol option ids. `ReplyInteractive`
  carries only the archived message id and visible button/choice indices;
  `interactive/replies.rs` re-resolves them from raw protobuf and uses the library's
  quote context and normal send path. Only known quick replies and single-select
  lists may send responses. Copy-code actions stay local. Do not turn arbitrary
  flow JSON into replies or fall back to sending its visible label as plain text.
  The versioned archive backfill must preserve downloaded image paths and edits.
  Carousel cards retain independent images and local actions. Download commands
  carry an optional card index, and the archive stores each image path separately.
  The version-4 backfill preserves those paths when rebuilding derived content.
  See [compatibility notes](docs/interactive-message-actions.md) for response
  families, source references, and live-test limits.
- Poll creation, voting, and decryption use whatsapp-rust's `Client::polls()`.
  `backend/worker/polls.rs` retains the original creator identity and key in the
  encrypted archive; `archive/polls.rs` keeps each voter's latest timestamp and
  message id, including encrypted updates whose parent has not arrived yet.
  History replay must not undo a newer vote or withdrawal. Decryption runs in
  batches of eight, with failures retried after reconnecting. The interface receives
  option counts, its own selection, and the latest decrypted voter names/times

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crmne/zapfast](https://github.com/crmne/zapfast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->

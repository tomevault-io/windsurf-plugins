---
trigger: always_on
description: validates them; keep it that way (audit #7). `automation=on` is what lets
---

# Blip — agent notes

Blip is an Omarchy (QuickShell/QML) bar plugin that puts iMessage on Linux by
treating a Mac as the gateway. Read this before touching anything.

## Shape

```
(Mac side)    bridge/mac/           VENDORED from claude-on-mac (pin in bridge/BRIDGE-VERSION; refresh with
              imsg imsg-send        scripts/sync-bridge.sh <rev>). Installed to ~/.blip/bin on the Mac by
              contacts tcc-check    bridge/mac/install.sh. Blip is ONE source for release (Fred's rule).
              blip-dispatch         forced-command gate for ~/.ssh/blip_ed25519: only the five tools run.
                                    imsg: sqlite read of chat.db, `--rich` (tapbacks/read_at/reply_to/
                                    attachments/error), `watch`, `attachment`, `chats`; Recently Deleted hidden.
(Linux side)  bridge/linux/blip-shim installed as ~/bin/{imsg,imsg-send,contacts} by scripts/blip-setup;
                                    reads ~/.config/blip/bridge.conf (host=, remote_bin='$HOME/.blip/bin'
                                    — single-quoted, expands on the MAC). `ssh -n` preflight; exit 69 offline.
                                    Blip only ever calls ~/bin/imsg*. No hostnames in code, ever.
collector.ts                        poll → {threads, unread, toast}. Pure functions + one spawn.
thread.ts                           one conversation → decorated bubbles. Pure + one spawn.
fetch.ts                            attachment id → ~/.cache/blip/att (0700/0600, 500MB LRU).
send-file.ts                        local file + caption → imsg-send --file-stdin --text-stdin-bytes N
                                    (caption ahead of the bytes; NO message text in argv anywhere). Resolves
                                    group guid from state; REFUSES unknown groups.
avatar.ts                           handle → ~/.cache/blip/avatars (imsg avatar; JPEG/PNG magic
                                    checked; .none negative marker; 7-day TTL).
paste.ts                            clipboard snapshot → draft image in $XDG_RUNTIME_DIR/blip or text.
BarWidget.qml                       the single poller, badge, toasts, IPC.
Panel.qml                           list view + conversation view + compose. Renders only.
manifest.json                       plugin id nixfred.blip
```

Logic lives in TypeScript where it can be unit-tested (`bun test`). QML renders
what it is handed. Keep it that way.

## Invariants — do not break

- **Never send to a handle for a group.** A group thread's `handle` is whichever
  member spoke last. `isSendable()` tests the *chat* id; groups send
  `--chat-id <full guid>` (`any;+;<32hex>`), DMs send `--to <chat>`.
- **Two read marks.** `watermark` = what the collector has seen (drives toasts).
  `readMark` / `readMarks[chat]` = what the user has looked at (drives the badge
  and the blue dots). Collapsing them makes the badge flash and reset.
- **Unread = BOTH sides agree** (1.3.2): Apple-side `is_read`=0 (imsg ≥1.9.0
  `read`; phone-synced via Messages in iCloud) AND newer than the local mark.
  Phone-read clears Blip within a poll; Blip-read clears locally only.
  Tapback rows and the self-thread never count (no Apple client badges them).
  chat.db carries GHOST is_read=0 rows years old — never trust is_read alone.
- **Read marks are per-chat, clamped to now, and --seen-based.** A message
  can carry a FUTURE timestamp (tz skew); a mark taken from the global max
  once suppressed unrelated threads until "tomorrow". The panel passes the
  newest VISIBLE ts (`--seen`) so mid-round-trip arrivals stay unread.
- **Reads are optimistic-with-suppression.** Persistent read state moves only
  via collector runs (~1 s), so BarWidget applies reads to the local model
  IMMEDIATELY and remembers them in `localReads[chat]` (thread last_ts at
  read time, 60 s TTL). Every collector result is filtered through that
  ledger — a poll in flight when the user opened a thread must not resurrect
  the dot for one round-trip. A chat only shows unread again when a NEWER
  inbound exists. Refreshes carry `activeReadChat()` so a message landing in
  the conversation being READ is counted read in the same run — never
  flashed. Same-chat read refreshes coalesce in the queue.
- **Unread is ledger-backed.** `unreadCounts` and `unreadOldest` persist per-chat
  metadata without message bodies. Catch-up fetches cover new arrivals and the
  oldest outstanding unread so deletions are reconciled; never derive the total
  badge solely from the preview window.
- **Dedupe the self-thread before counting.** A message you send yourself lands
  twice (`from_me` true and false, same ts+text). `dedupeSelfEcho()` runs before
  `buildThreads()` and before `decorate()`.
- **`PanelKeyCatcher` eats keys before focused children.** Any editor that
  should receive typing must be covered by its `blocked:` binding.
- **Message text never rides argv, on either machine.** Text: `--text-stdin`.
  Caption + file: `--text-stdin-bytes N` ahead of the bytes, and BlipView
  hands send-file.ts the caption via `--caption-stdin`. `--file-bytes N`
  makes the Mac refuse a short stream. `--keep-dashes` always (imsg-send's
  dash scrubbing is a claude-on-mac house rule, not Blip's).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nixfred/blip](https://github.com/nixfred/blip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->

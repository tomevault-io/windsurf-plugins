---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Unified Multi-Platform Messaging Client — Full Specification

## Quick Reference — Build & Test Commands

```bash
# Enter dev shell (required — provides Go, Flutter, Android SDK, all deps)
nix develop

# Build Go shared library
scripts/build_go.sh linux     # → go/build/libcores.so
scripts/build_go.sh windows   # → go/build/cores.dll (needs mingw32-gcc)
scripts/build_go.sh darwin    # → go/build/libcores.dylib
scripts/build_go.sh android   # → go/build/android/*/libcores.so (needs ANDROID_NDK_HOME)
scripts/build_go.sh web       # → go/build/cores.wasm + wasm_exec.js

# Shell aliases (defined in flake.nix)
build-go                      # → scripts/build_go.sh (current platform)
test-go                       # → cd go && go test ./...
test-dart                     # → cd dart && flutter test
test-all                      # → test-go && test-dart

# Run unit tests (no credentials needed)
cd go && go test ./utils/... -v -timeout 120s    # 91 tests, all passing

# Run a single test
cd go && go test ./cores/... -run "TestTelegramBotAuth" -v -timeout 30s

# Integration tests (need env vars from auth/auth.md)
source auth/auth.md && cd go/tests && go test -v -timeout 300s
```

**Go version**: 1.26.1 · **CGO_ENABLED=1** (for bridge only, cores/utils are pure Go)

## Current State (updated 2026-04-09)

**GitHub repo:** https://github.com/DarkReaperBoy/uniclient (HTTPS push, no SSH)

**Read these files first on context reset:**
1. This file (`CLAUDE.md`) — full spec and rules
2. `CHECKLIST.md` — what's done, what's next
3. `README.md` — public project overview (keep in sync with progress)
4. `auth/auth.md` — test credentials (bot token, chat IDs)
5. `docs/telegram_notes.md` — gotd/td API quirks and bot limitations

**What's done:**
- Phase 0 complete: flake.nix, 10 Go utils (91 tests), Core interface (`base.go` 320 lines), FFI bridge (`bridge.go` with JSON-encoded BridgeResponse, event port callback)
- Phase 1 Telegram ~95% (bot + user mode): `telegram.go` 743 exported methods, 8695 lines
  - Bot mode: 35+ features tested and confirmed
  - User mode: 22/22 verification tests + 20+ two-user tests pass against live API (2026-04-05)
  - Features: auth, profile, dialogs, messages CRUD, reply, forward, react (needs premium), pin/unpin, search, mark-as-read, typing, folders, contacts, scheduled messages, stickers, active sessions, file upload+download (byte-identical), drafts, online status, config, polls, translate, chat invites, channel management, stories, bot interaction, forum topics, channel admin toggles
  - Two-user tests verified: real-time updates (new/edit/delete), outbox read date, mark dialog unread, invite to channel, forum CRUD, basic group add/remove, poll votes, stories send/delete, import contacts
  - 600+ passthrough MTProto API wrappers for direct protocol access
  - Calling: **9/9 versions pass automated C++ harness** — pure Go/pion, no .so dependency. See `docs/tgcalls_protocol.md`
    - **V2Reference**: v10.0.0 (98 frames), v11.0.0 (98 frames) — SDP offer/answer, DTLS-SRTP
    - **V2Impl**: v7.0.0 (65), v8.0.0 (65), v9.0.0 (65), v12.0.0 (66), v13.0.0 (65) — InitialSetup+NegotiateChannels, synthetic SDP
    - **InstanceImpl**: v5.0.0 (52 frames), v2.7.7 (51 frames) — binary typed messages, AES-CTR encrypted raw ICE (pion/ice.Agent), no SDP/DTLS
    - Version ordering: `[11.0.0, 10.0.0, 13.0.0, 12.0.0, 9.0.0, 8.0.0, 7.0.0, 5.0.0, 2.7.7]`
    - V1 + V2 encryption framing, SCTP transport, NC dedup (`answeredExchangeIDs`), SSRC echo fix
    - **Real tgcalls C++ test harness**: libtgcalls_native.so + nix tg_owt, automated `TestAllVersions` via CGo
    - Config flags: `ForceV2Sig`, `ForceV2Impl`, `ForceV2Ref`, `ForceV1Framing`, `ForceInstanceImpl`, `ForceRelayICE`
    - Pure Go: MTProto lifecycle, DH exchange, V1+V2 signaling encrypt/decrypt, SCTP, pion WebRTC + pion/ice
    - Group calls: NOT IMPLEMENTED (SFU-based, separate implementation needed)
- Phase 2 (test server) architecture done: same code with `UseTestDC: true`, separate session at `auth/user_session_testserver.json`. Connects to test DC verified. Needs OTP for full smoke test.
- **Telegram Dependencies**: `gotd/td` (MTProto), `pion/webrtc` v4 (ICE/DTLS/SRTP/RTP), `pion/ice` v4 (raw ICE for InstanceImpl). All pure Go, no CGo.

**Second user account:** @dohix (ID 5435067494, +989204059095), session at `auth/telegram_user2_session.json`

- Phase 3 Bale: **COMPLETE** — `bale.go` 145 exported methods, 4092 lines
  - Bot mode: 17/17 verified. User mode: 68/68 verified, audited 1:1 with aiobale.
  - Calling: LiveKit-based (`bale.meet.v1.Meet`), **fully implemented, UNTESTED** (meet-em.ble.ir geo-restricted to Iran). 1:1 + group calls: signaling, LiveKit WebRTC via `livekit/server-sdk-go/v2`, silent audio track, remote track subscription, mute/unmute. Known: SDK sends `sdk=go` in WS URL, Bale may expect `sdk=js`.
  - File upload/calling geo-restricted (siloo.bale.ai, meet-em.ble.ir unreachable from abroad).
  - CreateChannel stubbed (use CreateGroup + SetRestriction). CreateTopic/Folders return ErrNotSupported.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DarkReaperBoy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

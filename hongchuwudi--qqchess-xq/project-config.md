---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Reverse-engineering toolkit for Tencent QQ Chess (天天象棋) H5 game's network protocol. Captures, decodes, and analyzes the WebSocket communication between the Cocos Creator H5 client and Tencent's servers.

## Commands

```bash
# Offline HAR analysis — extract WS messages, moves, FENs from a captured session
python har_analyzer.py data/h5login.qqchess.qq.com.har

# Live proxy — mitmproxy addon intercepting QQ Chess traffic in real time
mitmweb --listen-port 8888 -s xq_ws_proxy.py

# Decode individual base64 JCE messages
python xq_decode.py <base64_string>
python xq_decode.py --interact        # interactive REPL
python xq_decode.py --file msgs.txt   # batch decode

# Chess analyzer with AI engine integration
python xq_analyzer.py --har data/h5login.qqchess.qq.com.har
python xq_analyzer.py --demo           # simulated game demo
python xq_analyzer.py --proxy          # proxy mode

# Download the QQ Chess H5 client source from Tencent CDN
python download_qqchess.py

# Electron wrapper — one-click proxy + game launcher (no manual Windows proxy needed)
cd electron-app && npm start
```

Dependencies (no `requirements.txt`): `mitmproxy`, `requests`. Install manually.

## Architecture

### Protocol stack (bottom-up)

1. **Transport**: WebSocket binary messages on `wss://wxlogin.qqchess.qq.com:443`
2. **Framing**: `[2B big-endian length][magic][route string][JCE body]`
   - SEND magic: `01 10 cf XX YY` (5 bytes — first 3 fixed, last 2 = session identifier)
   - RECV magic: `0c XX YY` (3 bytes — first 1 fixed, last 2 = session identifier)
   - Session bytes vary per connection (QQ login: `10 01`, WeChat login: `1c 2c`, etc.)
   - Implementation: `unwrap_ws()` in `xq_modules/move_utils.py` uses prefix matching (not exact match)
3. **Encryption**: TEA-CBC (16 rounds, delta=0x9E3779B9), 128-bit key, with random head/tail padding. `iFlag & 1` on each message indicates encrypted.
4. **Serialization**: JCE (Jce Communication Encoding), Tencent's proprietary binary format — similar to Protobuf with tagged fields
5. **Session key derivation**: Login response 85001 delivers `sSecKey` + `uUin`. Derive: `sessionKey = TEA_decrypt(hexToBytes(sSecKey), pad16(str(uUin)))`. WeChat login uses the same derivation — `uUin` is always from TResponseLogin field 1, never from OpenID.
6. **Board state**: Chinese Chess FEN (10 rows × 9 cols, uppercase=red, lowercase=black)

#### QQ vs WeChat login (TResponseLogin variants)

The server sends one of two TResponseLogin structures depending on login type:

| Field | QQ variant | WeChat variant |
|-------|-----------|---------------|
| 0 | iResultID | iResultID |
| 1 | **uUin** | **uUin** |
| 10 | **sSecKey** | bShowButton (boolean) |
| 11 | banEndTime | **sSecKey** |
| 15 | iRoundID | sWXGameSessionKey |

Key derivation is identical for both: `RFe(sSecKey, uUin)` — `str(uin)` padded to 16 bytes as TEA key.

`parse_login()` in `xq_modules/protocol.py` uses raw-body byte scanning (not sequential JCE reads) to find sSecKey at either field 10 or 11, avoiding JCE reader position corruption from type mismatches.

### Coordinate systems & conversion

Three coordinate spaces exist, and every move must be converted correctly between them:

| System | Rows | Columns | Description |
|--------|------|---------|-------------|
| **Raw bytes** (protocol) | 1-indexed, 1–10 | 1-indexed, 1–9 (left→right: a=1..i=9) | Server's binary encoding in vecMsgBody |
| **Raw UCI** (0-indexed) | 0-indexed, 0–9 | a–i (left→right) | `_raw_move()` output — server's original intent, no interpretation |
| **FEN** (target) | 0–9, Red 5–9 bottom, Black 0–4 top | a=left, i=right | Canonical board coordinate; used by Pikafish engine and demo board rendering |

**Conversion chain**: Raw bytes → `_raw_move()` → Raw UCI → `game_to_fen()` / `rawToFenUci()` → FEN

#### Why conversion is non-trivial

The game protocol encodes moves from the **mover's perspective** (mover's own pieces at rows 0–4), using Chinese chess column numbering (right-to-left: 九→一). But `_raw_move()` maps columns left-to-right (a=1, b=2, ..., i=9). This creates a column mismatch: a Chinese column 八(h) may be encoded as raw byte 2 (b).

Additionally, the server sometimes pre-flips rows (Red at 5–9, matching FEN) while leaving columns in raw encoding. The result: a given move can arrive in one of two **formats**, distinguishable by the first move's `from_row`:

#### Format locking (final solution, 2026-05-10)

The format is detected once on the first move of a game and **locked for the entire game**. All moves within a game use the same operation — `mover_camp` (red/black) does NOT affect which operation to apply.

| Format | First move `from_row` | Operation (all moves) |
|--------|----------------------|----------------------|
| **A** | ≤ 4 | **Flip rows**: `fr = 9 - fr`, `tr = 9 - tr` |
| **B** | > 4 | **Mirror columns**: `fc = 8 - fc`, `tc = 8 - tc` |

- **Format A**: Both red and black moves → flip rows (columns unchanged)
- **Format B**: Both red and black moves → mirror columns (rows unchanged)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hongchuwudi/qqchess-xq](https://github.com/hongchuwudi/qqchess-xq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->

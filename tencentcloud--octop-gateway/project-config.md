---
trigger: always_on
description: Navigation guide for AI coding agents working in this repository.
---

# AGENTS.md

Navigation guide for AI coding agents working in this repository.

The working directory and distribution are **`octop-gateway`**; the import package is `octop_gateway`.

## 1. Collaboration principles

> Favor caution over speed; trivial tasks may relax these rules. These principles complement [§12 Change workflow](#12-change-workflow) and [§13 Communication](#13-communication).

### Think before writing

- Read first — copy the existing channel with the closest protocol (Feishu/DingTalk for SDK callbacks, WeCom for streaming, Telegram for bot polling) instead of inventing a new abstraction.
- State assumptions up front; ask when unsure. When several interpretations exist, list them and let the user choose — do not pick one silently.
- Check scope — bug fix, new channel, and refactor each have different success criteria (see [§12](#12-change-workflow)).
- Push back when a simpler approach exists.

### Simplicity first

- Write the minimum code that solves the problem; no unrequested features, abstractions, or config knobs.
- No defensive error handling for scenarios that cannot realistically happen.
- Match the existing channel style even when you would do it differently.

### Surgical edits

- Touch only lines directly related to the task; do not opportunistically "clean up" nearby code, comments, or formatting.
- Do not refactor unrelated broken code — mention it, do not fix it unless asked.
- Remove orphan imports, variables, and functions **you** introduced.
- **Channel-specific:** inside subclass methods call `self._send_*` only — never `self.reply_*` / `self.push_*` (throttle hygiene, see [§8 Outbound API](#outbound-api-reply--push--_send)).

### Verifiable outcomes

| Task | Plan | Verify |
|------|------|--------|
| Bug fix | Reproduce with a failing test → fix → run the suite | `make test` passes; the new test fails before the fix |
| New channel | Copy the closest existing channel → implement required methods → tests + example | `tests/channels/test_{name}.py` passes; `make lint` clean |
| Behavior change | Add the test for the new behavior → implement | targeted `pytest tests/...`, then `make test` |
| Refactor | `make test` before → refactor → `make test` after | zero behavior change unless requested |

**Ship bar:** `make all` (format + lint + typecheck + test) — exactly what the pre-commit hook runs. Enable the hook once per clone with `make install-hooks` (see [§6](#6-run-commands)).

For multi-step work, state a short plan with verify steps:

```
1. Read feishu.py media upload pattern → verify: understand _send_media flow
2. Implement _send_media in {name}.py → verify: pytest tests/channels/test_{name}.py -k media
3. Run the full suite → verify: make test && make lint
```

## 2. What this is

Multi-platform IM channel bridge with a unified message abstraction for AI agents and bots.

Core pattern: a `MessageProcessor` (async generator) feeds events through a `BaseChannel` managed by `ChannelManager`.

Built-in channels: Discord, QQ, Feishu (Lark), DingTalk, WeCom (Enterprise WeChat), WeChat iLink, Yuanbao, Xiaoyi, MQTT, Telegram.

| Platform | Channel kind | Transport | Text | Media |
|----------|--------------|-----------|------|-------|
| Discord | `discord` | Gateway WebSocket + REST | ✅ | ✅ |
| 飞书（Lark） | `feishu` | WebSocket + REST | ✅ | ✅ |
| 钉钉 | `dingtalk` | Stream | ✅ | ✅ |
| QQ | `qq` | WebSocket | ✅ | ✅ |
| 企业微信 | `wecom` | Callback + API | ✅ | ✅ |
| 微信 iLink | `weixin` | HTTP long-poll | ✅ | ✅ |
| 元宝 | `yuanbao` | HTTP REST | ✅ | ✅ |
| 小艺 | `xiaoyi` | WebSocket | ✅ | ✅ |
| MQTT | `mqtt` | MQTT | ✅ | ✅ |
| Telegram | `telegram` | Long-polling | ✅ | ✅ |

## 3. Tech stack

| Layer | Technology |
|-------|-----------|
| Language | Python 3.12+ — `from __future__ import annotations` in every source file |
| Async runtime | `asyncio`; SDK callback threads must re-enter the loop via `self.enqueue()` |
| Data models | Pydantic v2 (`BaseModel`, discriminated unions) |
| HTTP | `aiohttp` (channel HTTP), `httpx` where a platform SDK pulls it in |
| Platform SDKs | `lark-oapi`, `dingtalk-stream`, `discord.py`, `python-telegram-bot`, `wecom-aibot-sdk`, `paho-mqtt`, `websockets`, `python-socketio` |
| Packaging | hatchling; `uv` for local dev (`uv sync --extra dev`) |
| Quality gates | ruff (lint + format, line length 120), mypy strict, pytest + pytest-asyncio |

## 4. Package layout

```
src/octop_gateway/
├── models.py          # InboundMessage, MessageEvent, ContentPart, ChannelSubject, GroupContext
├── channel.py         # BaseChannel ABC + ChannelConfig + shared inbound pipeline + media flow
├── manager.py         # ChannelManager (queue, workers, session batching, push, add_*_channel)
├── constraints.py     # ChannelConstraints, RateLimiter, ReplyTimeoutGuard, TypingKeepalive
├── group_context.py   # Group activation policy + short-lived passive-chatter buffer
├── push_routing.py    # Proactive (bot-initiated) push metadata helpers
├── media.py           # MediaBackend ABC + FileSystemMediaBackend
├── utils.py           # Debouncer, content helpers
└── channels/          # one module per platform; lazy-loaded via channels/__init__.py
    ├── discord.py  feishu.py  dingtalk.py  wecom.py  mqtt.py
    ├── qq/            # channel.py + stream.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TencentCloud/octop-gateway](https://github.com/TencentCloud/octop-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->

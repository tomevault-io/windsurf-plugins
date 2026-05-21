---
trigger: always_on
description: Wxview is a local-first CLI for reading data from the user's own macOS or
---

# Agent Notes for Wxview

Wxview is a local-first CLI for reading data from the user's own macOS or
Windows WeChat.
The V1 implementation is intentionally narrow: initialize supported DB keys,
decrypt `contact/contact.db`, `session/session.db`, message-related DBs, and
selected optional data DBs, run a local daemon for cache maintenance, list
contacts/groups from the decrypted contact cache, query recent/unread/incremental
sessions, and query history for an explicit username from decrypted
`message/message_[number].db` caches. It also supports bounded message-content
`search` and cross-conversation `timeline` queries for AI and script consumers.
Image, video, file, voice, and local avatar media should resolve usable local
paths automatically when the supporting caches are available.

## Collaboration Rules

- Do not run git operations such as commit, push, rebase, or branch changes
  unless the user explicitly asks for git work.
- Do not print full DB keys, raw secrets, or unnecessary WeChat data. Printing a
  short key fingerprint is acceptable.
- `wechat-decrypt/` is an external reference repo and is ignored by the root git
  repo. Read it for behavior, but do not mix it into the Go runtime path.
- Runtime state under `~/.wxview/` is local user data and must not be committed.

## Command Semantics

- `wxview init` is the first-time setup command. It detects WeChat, finds or
  reuses required DB keys, verifies each against page 1 HMAC, and saves them
  in `~/.wxview/cache/<account>/keys.json`. Normal use should run it once at
  the beginning. Auxiliary message DB keys are best-effort and should warn
  rather than block init.
- `wxview init` should be concise by default: print account, data_dir, key
  counts, and warnings. Per-DB fingerprints/status belong behind `--verbose`.
- On Windows, account discovery reads `%APPDATA%\Tencent\xwechat\config\*.ini`
  and searches `xwechat_files\<account>\db_storage`. `WXVIEW_WECHAT_DB_STORAGE`
  may point directly at a `db_storage` directory when auto-detection fails.
- Current account detection should prefer the account whose `db_storage` files
  are currently open by the running WeChat process. File mtime is only a
  fallback when open-file detection cannot identify an account.
- `wxview contacts` with no flags must show the same help as
  `wxview contacts --help`; it should not query data. Require an explicit
  output/filter flag such as `--format` or `--kind` to query.
- `wxview contact` is a supported alias for `wxview contacts`.
- `wxview contract --help` is accepted only as a typo-friendly help alias. Do
  not make `contract` an official command.
- `wxview contacts` is intended to be usable by other tools and AI agents. It
  supports `--format table|json|jsonl|csv`, `--kind`, `--query`, `--username`,
  `--limit`, `--offset`, `--sort username|name`, and `--count`. Prefer `json`,
  `jsonl`, or `csv` plus an explicit `--limit` for automated reads.
- `--count` reports the filtered total before pagination; `--limit` and
  `--offset` do not affect the count.
- `wxview messages` requires `--username`; without args it must show the same
  help as `wxview messages --help`. It supports `--format table|json|jsonl|csv`,
  `--date`, `--start`, `--end`, `--after-seq`, `--limit`, `--offset`,
  `--source`, and `--refresh`.
- `wxview messages --format json` returns a `{meta, items}` envelope. `meta`
  includes `schema_version` and `timezone`. Reliable machine pagination should
  follow `meta.next_args` / `meta.next_after_seq`; `--offset` is mainly for
  temporary human reads. `jsonl`, `csv`, and `table` remain item-only outputs.
- Treat the `--username` value as an ordinary chat target even when it matches
  the current account username. Do not add a special self-chat guard or require
  an extra override flag for that case.
- `wxview messages` returns records sorted by time ascending by default. Apply
  `--limit` and `--offset` after merging all matching message shards.
- V1 maintains a derived local message index when available. `messages` may use
  it as a fast path when the index is ready, covers the chat, and is either
  globally near-real-time or chat-level fresh according to `session/session.db`.
  It must fall back to direct decrypted message-cache scanning when the index is
  missing, building, incompatible, or stale for the requested chat.
- `--start` and `--end` are inclusive. Date-only `--end` includes the full day.
- `messages --date today|yesterday|YYYY-MM-DD` selects one local calendar day
  and is mutually exclusive with `--start`/`--end`.
- `--after-seq` is a cursor-style filter. It returns only rows whose `seq` is
  greater than the provided value, then global sorting and pagination still
  apply.
- `--source` is a debugging flag only. It may expose source DB/table/local row
  metadata for cache/shard diagnosis, but normal chat-history reads should not
  require it.
- `wxview timeline` selects one or more conversations and returns a bounded
  cross-conversation time range. It supports `--format table|json|jsonl|csv`,
  `--kind all|friend|chatroom|other`, `--query`, `--username`, `--date`,
  `--start`, `--end`, `--limit`, `--cursor`, `--source`, and `--refresh`.
- `timeline` may use the derived message index only when it covers every

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yinwm/wxview](https://github.com/yinwm/wxview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->

---
trigger: always_on
description: Claude has independent local-usage/session and account-limits planes. A Claude Code transcript is not proof of which Claude account owns a limits row, and a Web or OAuth credential is not a token-history source.
---


# Claude

Claude has independent local-usage/session and account-limits planes. A Claude Code transcript is not proof of which Claude account owns a limits row, and a Web or OAuth credential is not a token-history source.

## Usage and session metadata

Aggregate tokens and costs come from tokscale. The local adapter in `src/shared/providers/claude/sessionMetadata.js` enriches sessions from `CLAUDE_CONFIG_DIR` or `~/.claude`, checking `projects/` before `transcripts/`.

It reads only persisted `custom-title` and `ai-title` records; it never turns prompt text into a title. Custom titles win. The index is keyed by file size and mtime, scans appended bytes after the first pass and keeps reads bounded around oversized JSONL records.

Turn state comes from the newest assistant `stop_reason` plus any genuine user prompt written after it. `tool_use` is not an ended turn. `tool_result`, meta and compaction records are not new user prompts. The adapter emits `true`, `false` or no value deliberately: `false` must clear an older finished state, while no value means there is no evidence.

### Live context occupancy

Claude emits `contextTokens`/`contextWindow` from its transcript for sessions that pass `shouldReadSessionContext()`. Use the last serving `message` or `fallback_message` iteration when present, not the request's cumulative usage rollup. Occupancy is `input_tokens + cache_creation_input_tokens + cache_read_input_tokens`; `output_tokens` is not part of the current input context. `input_tokens` is required, while missing cache counters mean zero. A malformed present counter, zeroed assistant notice or incomplete usage record must not erase the last valid reading.

An explicit `[1m]` model marker, supported native Claude 1M model ids and recognized provider Sonnet 5 ids map to 1M. Other nonempty model ids map to Claude Code's 200K default, including third-party bridges as a best-effort estimate; do not present that fallback as an authoritative bridge window. A compact boundary or compact summary clears the pair to `0/0` until the next valid assistant response. Oversized JSONL records use bounded head/tail fragments; accept context only from a complete message-level `usage` object and do not mistake nested `model` or `usage` fields in tool content for the response.

## Limits source order

`fetchClaudeLimits()` uses these mutually exclusive paths:

1. an explicitly configured Claude Web `sessionKey`;
2. Claude Code OAuth credentials discovered from env/file, Windows Credential Manager or macOS Keychain;
3. the authenticated Claude CLI usage screen as a fallback only for not-configured, rate-limited, unavailable or generic OAuth failures.

An identity-resolution failure after successful OAuth quota is not allowed to fall through and mint a differently keyed CLI row. The limits runtime retains the previous stable account instead.

OAuth usage refreshes reactively after unauthorized responses. Non-macOS platforms may also refresh shortly before expiry; macOS avoids proactive delegated refresh because it spawns Claude Code. Windows credential-file discovery includes running WSL homes when no explicit config root overrides it.

## Usage-limit reset grants

Both usage responses carry Anthropic's reset coupons in a `cedar_ember` block only when the request asks for it, so every OAuth and Web usage call sends `cedar_ember=1` — the same request, not an extra fetch. Grants that still hold resets and have not reached `ends_at` map onto `provider.resetCredits` with per-grant detail (label, cleared windows, usability flags); spent and lapsed grants are dropped so the count never promises a reset the account cannot use.

The OAuth endpoint also gates the block on client surface: a user-agent that is not Claude Code gets `eligible: false` with `ineligible_reason: "surface"` and no grants, so the OAuth usage call presents as `claude-cli/<version> (external, cli)` — the credential is a Claude Code token. Web usage calls are unaffected because they already present as the browser.

`seven_day_overage_included` in a grant's `clears` reads like a modifier on the weekly clear, but Claude Code's own label map calls it "Fable limit" — the credits-backed model's weekly bucket. Most accounts never see that bucket, so it folds into the general Weekly entry when `seven_day` is also cleared and lists as `Fable weekly` only when it is the sole weekly coverage. The display is read-only: the claim endpoint is deliberately not wired, so nothing in the app can spend a grant. A second reset program (`juniper_tide`) exists but is not collected.

## Claude Web

The stored value must be one bare `sk-ant-…` session key or canonical `sessionKey=…`; arbitrary Cookie headers are rejected. Web collection has priority when configured and uses Electron's dedicated native request adapter. That adapter preserves raw `Set-Cookie` headers and aborts the underlying request, which ordinary fetch handling cannot guarantee here.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Javis603/token-monitor](https://github.com/Javis603/token-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

---
trigger: always_on
description: Structural walls enforce these at write time (`.rules/kotlin-splice/` for the
---

# AGENTS.md — contracts for anyone (human or agent) touching splice

## The invariants (L1 retired; L2–L4 locked)

Structural walls enforce these at write time (`.rules/kotlin-splice/` for the
gateway, `.rules/rules/` for the webui, orchestrated by
`.claude/hooks/orchestrator.py`) and permanent tests enforce the behavioral
half (the `gateway/` module suites, plus the migration oracle's 11 byte-exact
fixtures — `npm run oracle:replay`). Do not weaken either.

1. **L1 — hard lock RETIRED (2026-07-14); replay DEFAULT-OFF (2026-07-15,
   measured).** Was "no reasoning-item replay." Replay is now a supported,
   config-gated behavior (`replayReasoning`): `codex/translate-request.mjs` can
   set `include: ['reasoning.encrypted_content']` and decode `redacted_thinking`
   blocks back into reasoning input items. But it ships **OFF**: measured, replay
   makes the model REUSE its prior thinking instead of re-deriving fresh, which
   suppressed the detailed-reasoning "wall" — output fell ~4× and the reasoning
   went thin. So the default IS the pure distillation loop — **mirror on (L2),
   replay off** — and `prompt_cache_key` (keyed on the first user message) warms
   the INPUT cache independently of replay (66% hit, up to 99% on continuation,
   with replay off). Opt into replay for extra cache warmth over reasoning depth
   with `CLAUDEX_REPLAY_REASONING=1`.
2. **L2 — the mirror never regresses.** ONE `mirrorInto()` in
   `reasoning/mirror.mjs`, called by the stream path (`codex/stream.mjs`) and
   the non-stream path (`codex-proxy.mjs`). Thresholds are named there:
   mirror ≥20, promote ≥40, honesty <20 chars.
3. **L3 — honest failures.** `anthropic/sse.mjs` is the only module that can
   emit `message_stop`; clean stops only via `emitTerminal(...)` (which owns
   stop_reason derivation), failures only via `emitError(...)`. A failed or
   truncated stream can never masquerade as a clean `end_turn`.
4. **L4 — no fake summaries.** Empty in, empty out. Promote-to-text promotes
   MODEL content (the reasoning summary) only; the only fabricated literals
   anywhere are error strings and `[… omitted …]` markers.

## External contracts (must not change)

- State paths byte-identical: `~/.claude-codex/state/*` and
  `~/.claude-codex/claudex-compact-stats.jsonl` (an out-of-repo HUD reads them).
- Config dirs keep their names: `~/.claude-codex`, `~/.claude-splice`.
- Ports: claudex 3099, control 3096. `/health` keeps `version`.
- Discovery prefix `claude-codex--`; the pinned model is excluded from
  `/v1/models` (it rides `ANTHROPIC_CUSTOM_MODEL_OPTION`).
- Effort precedence (v27): explicit body effort field, then the Claude
  `/effort` picker (`thinking.budget_tokens`), then config/env fallback, then
  `high`. Compact turns inherit the session's own model AND effort (a mismatch on
  either invalidates the prompt cache and re-reads the whole transcript cold);
  tools stripped.
- Mirror wire format: `\n[reasoning summary]\n<text>\n` (`mirrorWireText`).
- Reasoning replay envelope (`reasoning/replay.mjs`): encrypted reasoning rides
  as a `redacted_thinking` block tagged `splice-reasoning` v1; encode/decode
  stay paired (a tag/version bump strands in-flight transcripts).
- `prompt_cache_key`: `splice-<sha256(first user message)[:32]>`, stable per
  conversation. Changing the derivation cold-starts every live session's cache.

## Management API

> **Wire contract, Kotlin implementation.** The behaviour below is the contract; the `server/`
> Node tree that first implemented it was **deleted on 2026-08-10** (P8-CUT). The live sources are
> `gateway/control/.../ControlServer.kt`, `gateway/gateway/.../wire/SseEmitter.kt` and
> `gateway/gateway/.../reasoning/Mirror.kt`. Where this section still reads as prose about a
> `.mjs` file, treat the contract as authoritative and the filename as history — the 11 byte-exact
> oracle fixtures pin the wire itself.

Bearer-guarded (`Authorization: Bearer $(cat ~/.claude-codex/state/mgmt-key)`),
loopback-only, both proxies:

| route | purpose |
|---|---|
| GET /mgmt/status | version, uptime, gate snapshot + live in-flight, mode/arm |
| GET /mgmt/config | effective + all four layers + restart-required keys |
| PATCH /mgmt/config | hot-apply runtime knobs; persists to state config.json |
| GET /mgmt/usage | 5h output-token window + persisted ratelimit headers |
| GET /mgmt/compact | compact outcomes + shadow-classifier tail |
| GET /mgmt/auth, POST /mgmt/auth/refresh | token introspection (masked), refresh |
| GET /mgmt/logs?tail=N | proxy log tail from ~/.claude-codex/logs/ |
| GET /mgmt/models | catalog, windows, pinned, discovery ids |

Config layering: defaults ← `[defaults]` TOML ← `[heads.<key>.overrides]` TOML ←
state/config.json ← env ← runtime PATCH. Env is the boot authority (the launcher
writes it); PATCH wins until restart and persists to the file layer. `port`,
`grokPort`, `controlPort`, `upstreamTimeoutMs` need a restart; everything else
hot-applies on the next request.

All heads share ONE `ConfigService` (one JVM — unlike the Node lineage's
process-per-head), so a knob read via `getConfig()` governs EVERY head. Anything
that belongs to a single upstream account — `maxInflight`, `maxQueued`, the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [torad-labs/splice](https://github.com/torad-labs/splice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->

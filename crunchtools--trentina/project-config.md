---
trigger: always_on
description: Validates the whole file before swapping (a bad edit keeps the running
---

# trentina

MCP gateway between AI agents and everything they touch. Five promises: prompt-injection
defense (L1 ∥ L2 → L3 at every ingress), token savings, deterministic policy,
authentication (OAuth/DCR), and architectural flexibility. README.md is the
canonical statement of them; keep docs and comments in line with it.

## Quick Start

```bash
uv sync --all-extras
uv run trentina
```

## Environment Variables

- `TRENTINA_MODEL_PROVIDER` + that provider's key (`OPENROUTER_API_KEY`,
  `ANTHROPIC_API_KEY`, ...) — L3 when no gateway profile is bound. Behind the
  gateway each profile's `llm_keys` is used instead. `config.has_llm` and
  `agent.llm_available()` answer "is there an LLM"; `has_api_key` is Gemini's
  key only. Since 0.41.0 it no longer gates L3 or summarize; it only picks
  Gemini grounding as the standalone search fallback.
- `GEMINI_API_KEY` — optional since 0.41.0: Gemini as an L3 provider, and
  search grounding ONLY standalone (no profile bound) with no
  `OPENROUTER_API_KEY`; a bound profile searches on its own OpenRouter key or
  is refused. The lotor gateway
  holds none, so it never calls Google directly.
- `OPENROUTER_API_KEY` — standalone key for `TRENTINA_MODEL_PROVIDER=openrouter`;
  a gateway profile uses `llm_keys.openrouter`. Model ids are OpenRouter's
  (`google/gemini-2.5-flash-lite`).
- `QUARANTINE_MODEL` — Gemini model for Q-Agent (default: gemini-2.5-flash-lite)
- `QUARANTINE_SEARCH_MODEL` — L0 web search model, an OpenRouter id (default
  `google/gemini-2.5-flash`). L0 is OpenRouter's `web` plugin and nothing else
  (`_enforce_openrouter_search_quarantine`); the Gemini grounding route strips
  the `google/` prefix.
- `TRENTINA_REQUIRE_L2` / `TRENTINA_REQUIRE_L3` — default true: block and redact
  refuse when that layer is ABSENT. `false` turns absence into a warning; a
  partial read is never excused. `QUARANTINE_FALLBACK` was removed in 0.31.0
  and setting it fails startup.
- `QUARANTINE_CONTEXT_TOKENS` — what the L3 model reads in one call (default
  1000000). With `CLASSIFIER_MAX_TOKENS` it sets `Config.admission_tokens`.
  `QUARANTINE_MAX_CONTENT` was removed in 0.43.0 (#225); setting it fails startup.
- `QUARANTINE_DB` — SQLite blocklist path (default: ~/.local/share/trentina/trentina.db)
- `TRENTINA_BLOCKLIST_TTL_DAYS` — days a block refusal stays on the blocklist
  (default 30, floor 1). Expired rows stop counting at once and are swept
  hourly by `is_blocked` (#263), `SWEEP_BATCH` rows a pass (#295).
- `TRENTINA_AUDIT_RETENTION_DAYS` — days a `gateway_calls` row is kept
  (default 90; 0 keeps every row). Swept with the blocklist, from
  `is_blocked` and `record_gateway_call`, in batches (#295).
- `TRENTINA_FETCH_CONCURRENCY` — fetches in flight per profile (default 8).
  A fetch also has `client.FETCH_DEADLINE` (60 s) of wall clock, every hop
  and the body; `egress.open_guarded` requires a `deadline` (#295).
- `TRENTINA_PERIMETER_DB` — perimeter verdict store, a SEPARATE database from the
  blocklist (default: `perimeter.db` beside `QUARANTINE_DB`). See `perimeter_db.py`
  for why it is its own file. Deleting it costs one slow restart and nothing else.
- `QUARANTINE_TRUST_CONFIG` — Trust allowlist JSON path
- `TRENTINA_READ_ROOTS` — `os.pathsep`-separated absolute directories that
  `read_tool` and `dir_tool` may reach (#261); a relative entry fails startup.
  Unset behind a live gateway refuses EVERY path, and production leaves it
  unset on purpose: the gateway container has no agent workspace. Unset
  standalone keeps full reach. Either way `tools/confine.py` refuses `/config`,
  `/data`, `/proc`, `/sys`, `/run`, `/dev` and the directories of the two
  databases, the trust config and the live `profiles.yaml`. Refusals are a
  closed reason code, never the path. The open walks the resolved path one
  `O_NOFOLLOW` component at a time (#287); a FIFO or device is refused
  unopened, and a hard link to Trentina's own files is denied. The path is checked as written
  (normalized) BEFORE it is resolved, then again resolved; behind a gateway
  `not_found`/`denied_path`/`outside_read_roots` are one reason,
  `not_found_or_denied`, so a refusal is no existence oracle (#263). A
  confinement refusal is a `BlockedSourceError` with no alternatives
  (`confine.refused`), audited `blocked_defense` like egress's (#278).
- `TRENTINA_FETCH_ALLOW_PRIVATE` — default false. Lifts the egress guard's
  address rule so fetch can reach non-global addresses; scheme, port and
  redirect rules still hold. Warns at startup. See `egress.py` (#260).
- `TRENTINA_REQUIRE_HARDENED` — default false. On a network transport,
  `posture.py` reads `/proc/self` at startup and WARNs each containment gap
  (no-new-privileges, capabilities, seccomp, writable rootfs or import path,
  a secret from the environment rather than `_FILE`); true refuses to start
  on any. See `docs/deployment-hardening.md` (#268). The image sets
  `PYTHONSAFEPATH=1` so the working directory is never on `sys.path`.
  `GEMINI_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY` and
  `OPENROUTER_API_KEY` take `_FILE` forms too.
- `TRENTINA_RATE_LIMIT` — "off" disables limiting on the unauthenticated OAuth
  write paths (default on). An incident escape hatch, not a setting.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crunchtools/trentina](https://github.com/crunchtools/trentina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->

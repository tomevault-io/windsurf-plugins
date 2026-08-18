---
trigger: always_on
description: **`readme.md` is the specification. This file is how you stay inside it.**
---

# AGENTS.md — rules for any agent working in this repo

**`readme.md` is the specification. This file is how you stay inside it.**

If something you are about to write is not in `readme.md`, you are either drifting or
inventing. Stop and check. Section references below (§4, §9, …) point at `readme.md`.

---

## 0. Current state

**M1 (Skeleton) and M2 (Tier 1 collector + router + dedup) are complete and verified.**
In place on top of M1: the collector package (`base`, `router`, `runner`, `sources/{rss,
github_releases}`), `pulse.scheduler` tiered polling, cross-source dedup (`story_key.py`,
`dedup.py`, the `story_keys` table via revision `0002`), the Analyzer pickup guard
(`analyzer/runner.should_analyze`), and `embeddings.py`. The default test suite is offline
and keyless; `uv run pytest -m integration` exercises the real gates, including the
story-key race, against the Compose Postgres. An M2 follow-up added
`generator/tasks.draft_admission` — the fail-closed three-way draft-admission policy
(DRAFT / HOLD / KILL) enforcing the tier-2 confirmation requirement (§2 items 30–32)
before any Generator code exists to get it wrong — plus `publisher/notify.count_held_tier2`
(the age-bounded heartbeat counter, visible from M3 onward) and
`generator/tasks.sweep_held_tier2` (stale expiry + newest-first selection for M10's
confirm worker).

**M3 (Telegram heartbeat) is complete and confirmed on a phone** (four beats, 15 min
apart, 2026-07-31): `publisher/{telegram,notify,runner}.py` — Bot API client (long
polling), heartbeat with window counts + the held-tier-2 line, update acknowledgement
(callbacks plug in at M8). Post-M3 hardening from live logs: boot-time tier-1 feed
validation (4xx fails loudly), conditional feed fetches (ETag/Last-Modified, 304s skip
parsing, `cache_hits` in the cycle summary), per-request feed timeouts with per-feed
isolation, and bot-token redaction in every error path (client messages, unchained
tracebacks, and a formatter-level scrub).

**M4 (Agent harness) is built:** `agents/{base,contracts}.py` + `prompts/README.md`.
The bounded loop enforces the cap in code, times out between turns, validates output at
the boundary (exactly one retry with the error appended), writes every run — including
failures and budget-skips — to `agent_runs` with `raw_output` captured before parsing,
and returns empty when the daily token budget is spent. The model client is a Protocol;
tests drive it with stubs (no network, no keys). Prompt files land with their agents.

**M5 (Gatekeeper) is complete and verified.** 2026-07-31: v1 moved
entirely onto the Gemini free tier — `GATEKEEPER_MODEL`, `GHOST_MODEL`, and
`ANALYST_MODEL` now default to Gemini, kept as separate keys so any one can move back
to `claude-*` without code changes (readme §16 has the tradeoff and the rationale).
`agents/clients.py` holds both real providers behind the `ModelClient` protocol,
dispatched on the model-name prefix; 429s are retried with exponential backoff
(`MODEL_MAX_RETRIES`) and exhaustion degrades like the token budget —
`AgentResult.exhausted`, HOLD never KILL. `agents/gatekeeper.py` + `prompts/gatekeeper.md`
(v1) are in place with offline tests and the labelled 20-topic eval set
(`tests/fixtures/agents/gatekeeper_eval.json`, `uv run pytest -m eval`). **M5's
"Done when" passed live 2026-07-31: 20/20 classified, tragic headlines blocked, p95 under
the 3s budget.** Re-run the eval on every gatekeeper prompt version change. Note the eval
judges against its fixture's fixed profile (its labels are only valid there), never the
`.env` `COMPANY_PROFILE`; for eyeballing the real profile's filtering there is
`uv run python -m pulse.agents.inspect_gatekeeper` — newest topics, real profile, printed
verdicts, no assertions, runs kept out of `agent_runs`. Tool-call translation in the
clients is deliberately absent until M10.

**M6 (Writer) is complete and verified.** `agents/writer.py` + `prompts/writer.md` (v3 —
2026-08-01 added one rule: `image_brief` derives from the written `post_text`'s argument,
not the topic headline alone, so the picture reflects what the post is about):
one Gemini call with `responseSchema`, no tools, no loop, through the same bounded
harness (`WRITER_TIMEOUT_SECONDS`, default 60). Input is the topic, the verdict's
`angle` (plus the Analyst's `summary` when the verdict is an `AnalystVerdict` — it read
the story; the collector's scrape didn't), the company profile, and §9's
`brand_voice_examples`, which live in the optional file at `VOICE_EXAMPLES_PATH`
(default `config/voice_examples.md`; no placeholder ships — fake examples steer tone
worse than none). Prompt v2 (2026-08-01): when voice examples are present they are the
authority on tone and formatting and override the prompt's style bans (emoji, bullets,
question closes) — the two must never fight; structure rules always hold: hook ≤12
words standing alone, then the news in ONE short human line ("OpenAI just published X",
never the input summary recited back), then the argued angle. The profile informs the
angle and nothing else — never listed, quoted, or paraphrased in the post, never our
own service lines named (that turns commentary into a services pitch, on every post).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rishabhfit20261/Pulse](https://github.com/rishabhfit20261/Pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->

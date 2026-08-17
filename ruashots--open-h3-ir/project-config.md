---
trigger: always_on
description: **This document is for changing the compiler.** Read it before changing anything. It is the set of
---

# Working on OpenH3-IR

**This document is for changing the compiler.** Read it before changing anything. It is the set of
rules that are not preferences, a map of which file owns what, and an honest list of what is missing.
There is no install path here, on purpose.

Two neighbours, if one of them is the job instead:

- **Installing it and making it run:** [HANDOFF.md](HANDOFF.md).
- **Calling the service from an application:** [docs/calling-the-api.md](docs/calling-the-api.md).

The project is `open-h3-ir`; the import package and the command are both `h3ir`.

The checks to hold your work against, both reproducible with no model and no GPU: `h3ir controls` is
21/21 in under a tenth of a second, and `pytest -q` is green. The control count is a gate and should
only move when you deliberately add or remove a control. The test count is not pinned here, because it
moves every time anyone adds a test and a stale number reads as a regression.

## What this is and where it runs

A local rebuild of MiniMax H3's closed Context-IR stage. Brief in, validated H3 prompt plus the
asset wiring it is true for, out.

- **Assume the compiler and the GPU are on different machines.** No path or URL is hardcoded
  outside `config.py`, and ComfyUI is always reached over HTTP, never through the filesystem. Keep
  it that way. A filesystem shortcut works on a single box and fails silently everywhere else.
- Reasoning and vision run on whatever `H3IR_LLM_URL` points at. Nothing calls MiniMax.
- `h3ir doctor` tells you what is actually reachable before you debug anything else.

## The rules that are not preferences

1. **Never let a model decide structure.** Labels, label order, speaker IDs, cut times, retention
   markers, task-type prefixes and section order are computed in `plan.py` and emitted by
   `render.py`. If you find yourself adding a structural instruction to a prompt file, the fix
   belongs in the planner instead.
2. **The user's words never pass through a model.** Dialogue reaches the output through
   `{{D1}}` placeholder substitution in `render.py`. If you change that path, `D4` will catch you.
3. **Never trust the endpoint's structured output.** It is documented in `backend.py` with the
   measurements: `json_schema` is silently not applied while reasoning is on, and even with it off
   the grammar constrains shape but not completion or sense. Parse and re-check everything.
4. **Never degrade silently.** If the model is unreachable the service raises. A caller cannot
   tell a good IR from a bad one, so quietly producing a worse one is the failure nobody notices.
5. **Nothing ships on judgement.** See below.
6. **The deterministic draft is the product floor, not a degraded mode.** `draft.py` builds a
   complete valid IR with no prose model. The LLM pass is additive. Any validator error, leaked
   reasoning, or model outage falls back to the draft, so the caller always gets something valid.
   The draft failing its own validator is the one thing that raises, because it is deterministic
   and there would be nothing to fall back to. Do not turn this back into retry-until-valid.
7. **Thinking is per call, not global.** ON for the beat sheet (planning: measured +5.3pp), OFF
   for extraction, classification and prose (precision: measured −8.5pp). This is contingent on
   code owning every machine-checkable field. If you ever let the model emit a timecode, turn
   thinking off for that call.
8. **Never enable guided decoding without re-reading why it is off.** vLLM #39130 can skip grammar
   enforcement silently with a reasoning parser active; llama.cpp #20345 reports the converse on
   this model family. `H3IR_GUIDED_DECODING=1` exists for comparison, not for production.
9. **The model is deaf. Never ask it about audio.** `analyse_audio` makes no model call. Audio
   facts are typed metadata plus a real transcript. An invented timbre is worse than none because
   `<Audio N>` carries no content into the encoder, so the IR text is its only channel.
10. **Proportionality is part of the bar, and it is an explicit input.** "If I ask for simple, I want
    simple, if I say go crazy, I want crazy." `brief.creativity` is `restrained | balanced | bold`,
    default balanced, and it governs exactly one thing: whether the writer may add **content the
    request never supplied**: a spoken line, a score, on-screen text. It does NOT mean "more shots"
    or "more camera moves"; putting effort on this dial would be shot-count-as-a-rule one layer above
    the validator, where nothing catches it. Never infer the setting from the request. That was
    considered and rejected, because it would be wrong often and the maintainer could not overrule it. An
    explicit prohibition in the request outranks every position: `bold` on "No dialogue" licenses no
    dialogue. See `creativity.py` and design doc section 19.
11. **A rule asserts a decidable fact, never a preference.** The test: could a competent director
    disagree with it? If they could, it is not a check, at any severity. Shot count is not a
    defect. Prose quality is not a defect. Whether an edit is good is the maintainer's call and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruashots/open-h3-ir](https://github.com/ruashots/open-h3-ir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->

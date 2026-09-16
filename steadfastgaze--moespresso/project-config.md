---
trigger: always_on
description: Read this before changing anything. It covers how to work in this repo and the
---

# Agent and contributor guide

Read this before changing anything. It covers how to work in this repo and the
invariants that must hold. For the architecture, see `DEVGUIDE.md` and `docs/`.

## Public product and fixture boundary

The public product models are DeepSeek-V4-Flash and Ornith. Source paths and
entry points containing `qwen` describe the Qwen architecture implementation
used by Ornith, plus engineering harnesses around that implementation. Their
presence does not add a third public product.

The DeepSeek-V4 fixture boundary is drawn by provenance, not by ladder rung.
Material the upstream antirez/ds4 suite published stays public and committed so
the gates run against those fixtures. Anything captured from a provider stays
under `src/moespresso/correctness/fixtures/deepseek_v4/private/`: API-derived
continuations, selected-token records, and top-logprob payloads, whether they
serve Q1 or Q2, together with the Q4 teacher and candidate logit dumps and the
marker set they are scored against. The WikiText corpus is third-party text
referenced by digest and never copied in. `docs/deepseek_v4_quality.md` is the
authority on which side a given fixture falls.

Unpublished Ornith benchmark questions, answer keys, and verification programs
remain under `src/moespresso/correctness/fixtures/ornith/private/`. Never copy
either private set into source, tests, logs, documentation, wheels, or source
distributions.

The public Ornith gate surface covers the project-owned agentic-coding and
long-context instruments:

```
uv run --locked moespresso-ornith-gate <package> \
  --families agentic_coding,long_context
```

The full hard-reasoning gate additionally requires the ignored private Ornith
fixtures. Source-release tests must inject synthetic questions and keys instead
of reading those files.

## Tooling discipline

- **Use `uv` for everything.** `uv sync` sets up the complete runtime and
  development environment. Tests and lint run lock-strict (`uv run --locked`), so an
  unlocked dependency edit fails immediately. `make lock` is the deliberate
  re-resolve step; commit the updated `uv.lock`. `make dist-check` builds and
  audits the public wheel and source distribution. Never call `python` directly
  and never `pip install` into the environment.
- **Search with `rg`.** Do not use `find ... -exec grep`.
- **Edit with a tool that fails loudly on a missing anchor.** A scripted patch
  that silently misses its anchor and writes nothing is the most common way a
  change appears done but is not. Prefer an editor that errors on a miss; if you
  must script an edit, assert the anchor.
- **No heredocs**, and do not chain shell commands with `&&` or `;` when each is
  already a separate allowed command. Run them as separate calls.
- **Put throwaway scripts in temporary files** and run them via `uv`.

## Writing comments and docs (this repo is public)

Every comment, docstring, and doc ships to a public GitHub repo. Write them the
way a large OSS project does: about the code, impersonal, and dateless. Write for
an unfamiliar reader. These mandatory rules came from real cleanup work.

- **No machine references.** Never "this box", "on this box", "my box/machine/
  laptop/mac", "the rig", "small box", "big box", "16 GB box". The computer is
  never the subject of a sentence. Say "the runtime", "the host", "under memory
  pressure", "when the system is idle".
- **No dates or dev-diary notes.** No "NOTE (2026-06-10):", "Measured on the rig
  (date):", "(owner decision 2026-06-12)". State the finding as a present-tense
  fact and keep the measurement number; drop the date and the first person.
  ("Extending the kick to all-hit layers measured flat (5.42 vs 5.48 tok/s), so
  the gate stays miss-only.")
- **No authority-by-person.** No "owner decision", "owner priority", "product
  priority (owner)". State the decision as a fact: "Default ON; `FOO=0` is the
  kill switch."
- **No private version archaeology.** "pre-v8 gibberish", "the v4 collapse" mean
  nothing to an outside reader. Name the *failure mode* instead: "storing conv1d
  pre-transposed suppresses the coupled norm shift, so norms load ~1.0 too low
  and the model emits garbage."
- **No shouting-caps for emphasis.** Not "THIS", "ONLY", "MUST", "NOT", "GARBAGE",
  "LOUD". Caps are for real identifiers and acronyms (MLX, GPU, IO, LFU, env-var
  names) only. Use plain words or `**bold**` in docs if you must stress a point.
- **No lazy LLM-ese in docs.** Avoid em dashes, antithetical definitions,
  parallel equal-length sentence pairs, and phrases such as "front door",
  "heart of", "by construction", "first-class", or "the tool you reach for".
  Use plain declarative sentences that state the mechanism. Say a fact once.
- **Leave program output and content strings alone.** Error messages, log lines,
  and calibration prompt strings are not comments; do not de-shout or reword them
  for tone.

## Invariants (do not break these)

These are properties the system depends on. Each was learned from a real
failure; respect them or prove with a measurement that the constraint no longer
holds.

**Pipeline and artifacts**
- Resolve once. The inventory maps every tensor name to a role a single time;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [steadfastgaze/MoEspresso](https://github.com/steadfastgaze/MoEspresso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->

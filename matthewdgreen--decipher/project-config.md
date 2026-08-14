---
trigger: always_on
description: Project context for Codex sessions. Keep this file updated as the project evolves.
---

# Decipher — AGENTS.md

Project context for Codex sessions. Keep this file updated as the project evolves.

---

## What This Is

A CLI research tool for classical cipher cryptanalysis. Primary focus:
- **Monoalphabetic substitution ciphers** with arbitrary symbol alphabets
- **Historical manuscripts** (Borg cipher in Latin, Copiale cipher in German)
- **AI-assisted decipherment** using Codex tool-use API
- **Benchmark evaluation** against a dataset of solved historical ciphers

Licensing note:
- Decipher is now GPLv3-licensed.
- `src/analysis/zenith_solver.py` is derived from the Zenith project by
  beldenge and must retain explicit attribution in code comments and docs.
- `src/analysis/homophonic_nulls.py` holds ground-truth-free null/codeword
  candidate generation and finalist validation helpers for homophonic ciphers.
  Shared by calibration scripts and the opt-in automated `null_masks`
  homophonic refinement profile. The automated null-mask route may also run
  a consensus-polish pass that freezes mappings agreed on by top finalist
  keys and reruns only disputed symbols; this consensus is solver-derived and
  must remain independent of benchmark plaintext. It can also optionally load
  a saved `LinearLanguageQualityModel` with
  `DECIPHER_NULL_MASK_LANGUAGE_QUALITY_MODEL` and
  `DECIPHER_NULL_MASK_RANKER=language_quality`; this is a ground-truth-free
  runtime ranker, but first-generation models are calibration aids rather than
  default Copiale evidence.
  The default automated null-mask profile is now the broad Copiale-style
  `wide` profile: it expands candidate/mask breadth, keeps a larger menu, and
  keeps a cross-ranker validation/LQ/ensemble finalist portfolio while
  still keeping benchmark plaintext out of candidate generation and selection.
  Use `DECIPHER_NULL_MASK_PROFILE=narrow` for quick reference/debug runs that
  intentionally reproduce the older compact search envelope.
  The expensive per-mask homophonic solves now batch through the mandatory
  Rust fast module by default (`DECIPHER_NULL_MASK_ENGINE=rust_batch`, threads
  controlled by `DECIPHER_NULL_MASK_THREADS` with the usual parallel-worker
  fallbacks). Python still owns candidate generation, validation/ranking, and
  artifacts, so the ground-truth firewall remains at the orchestration layer.
- The unchanged Zenith 2026.2 English binary model is redistributed at
  `models/ngram5_en_zenith.bin` and is the named `zenith_upstream` English
  default. Exact version, checksum, corpus sources, and the remaining Blog
  Authorship Corpus caveat are recorded in
  `docs/zenith_model_provenance.md`. Decipher-built English variants remain
  available, and improving them toward parity remains active work.

---

## Cracking a cipher (MCP quick path)

This repo ships an MCP server exposing the investigation surface. A checked-in
`.codex/config.toml` / `.mcp.json` wires it up once the project is trusted;
`sh scripts/bootstrap.sh` prepares a fresh clone. Methodology, tool
reference, and recovery live in **`docs/mcp_onboarding.md`** — read that
(not this file's development notes) when the task is *cracking a cipher*
rather than developing Decipher. After an investigation exists,
`investigation_status` is the authoritative briefing; do not treat onboarding
prose as live investigation state.

Repair doctrine: Distributed damage that is a set of individually-simple key errors is still batch-repairable via `repair_hypotheses_test` → `repair_transaction`; do not treat `distributed` automatically as broaden-only.

---

## Key Files

```
src/
  cli.py                  — CLI entry point (benchmark, crack, testgen subcommands)
  models/
    alphabet.py           — Alphabet class (symbol↔integer mapping, multisym support)
    cipher_text.py        — CipherText dataclass (raw text + alphabet + word structure)
    session.py            — Headless Session: cipher text, key dict, apply_key()
  analysis/
    frequency.py          — mono/bigram/trigram frequency, chi-squared
    ic.py                 — Index of Coincidence
    pattern.py            — Word isomorphs, pattern dictionary, match_pattern()
    dictionary.py         — load_word_set(), score_plaintext(), get_dictionary_path(lang)
    solver.py             — Algorithmic solver: hill_climb_swaps(), auto_solve()
    ngram.py              — N-gram language models with lazy caching
    polyalphabetic.py     — Vigenere/Beaufort/Gronsfeld solvers, keyed
                            Vigenere/Quagmire replay/search, and experimental
                            shared-tableau mutation search
    language_scoring.py   — Extensible language-profile signals for damaged
                            no-boundary plaintext ranking: coherence/shape,
                            word-lattice quality, content-word quality,
                            repetition, function-word overuse, and the
                            transparent trainable fast-scorer scaffold
    homophonic_nulls.py   — Null/codeword candidate generation and
                            ground-truth-free language finalist validation
    nomenclator.py        — Ground-truth-free symbol-level rendering helpers
                            for homophonic/nomenclator candidates: null masks,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matthewdgreen/decipher](https://github.com/matthewdgreen/decipher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->

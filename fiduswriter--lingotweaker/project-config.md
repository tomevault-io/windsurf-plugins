---
trigger: always_on
description: Commands for working in this repository.
---

# AGENTS.md

Commands for working in this repository.

## Build / test / lint

```sh
cargo check --workspace
cargo test --workspace
cargo fmt --all
cargo clippy --workspace --all-targets -- -D warnings
```

CI enforces all four. Tests read the vendored `data/` directory at the repo root;
without it, data-dependent tests skip themselves.

## Parity gate

```sh
scripts/oracle/gate.sh            # 2,000-example Java match-set gate (Docker)
```

Builds the sample from `docs/parity/corpora/en-examples.jsonl`, runs
`scripts/oracle/check-diff.sh` against the pinned Java checkout and fails
unless `only Java: 0; only Rust: 0; field diffs: 0`.

Gate scope: re-run another language's gate only when the change can influence
that language (shared engine/pipeline, `lt-pattern`, `lt-disambig`, tokenizer,
overlap filters or shared rule data); language-local changes only need that
language's gates/tests. The CI `parity` matrix enforces the same scope (P6.3,
D-134): the workflow computes the affected set with
`scripts/ci/affected-languages.sh` (language-local paths → that language,
shared/unrecognized paths → all, docs-only → none). Predict a push with
`scripts/ci/affected-languages.sh <base> <head>`; the filter's tests are
`scripts/ci/tests/affected-languages-test.sh`.

Divergence policy (D-310, owner 2026-09-23): fix a Rust/Java divergence only
when Java is more correct; where Rust is more correct, keep Rust and record the
divergence as intentional. Every `docs/differences.md` entry and every
`scripts/ci/parity.sh` allowance must state the correctness verdict and
distinguish **residue to fix** (Java-reference fidelity gap) from
**intentional: Rust more correct**.

Offline corpus gate (CI, no Docker):

```sh
cargo build --release -p lt-cli
scripts/ci/parity.sh en|de|es|fr|it|pt|nl|ca|gl|ro|pl|sk|sl|el|da|sv|is|eo|ast|br|tl|crh|be|ru|uk|sr|ar|fa|km|ml|ta|de-x-simple   # Java-golden languages
scripts/ci/parity.sh no|nrd|nn|gn|lt              # tests-only gate
```

Diffs the full per-language corpus against the pinned Java `CheckDump` goldens
in `docs/parity/golden/` (captured once with Docker); the CI `parity` matrix
runs the affected languages per push/PR (all gated languages when shared code
changed, no language job for docs-only changes). de/es/it/nl/ca/ro/sk/sl/el must
be 0 only-Java / 0 only-Rust / 0 field diffs; en allows exactly the one documented
`ADVERB_VERB_ADVERB_REPETITION` field diff, fr the documented divergences
#3/#4/#5 and pt #6; gl is at 0/0/0 (the former `HUNSPELL_RULE` = 2 suggestion
field diffs #7, resolved by the deterministic work-budget emulation of the
native hunspell suggestion timers: work is counted in affix-entry trials —
MAP-generated candidates weighted at a quarter, their measured per-check
trial cost being ~8x lower — with the per-pass `WORK_SUGGESTION` budget
checked at upstream's clock-check positions, so the two Galician words now
cut exactly where the pinned run's wall clocks did); pl is at 0/0/0 (the
former known fidelity gaps #9, resolved by the
`<unify>` engine fixes: negated/unified `<unify>` matching incl. per-token
reading sets for `max`-run elements, antipattern matching with the unifier and
marker spans, the `<match no="N">lemma</match>` disambiguation filter and the
Java possessive-quantifier regex semantics); da is at 0/0/0 (#10, resolved by the
suggestion-engine and dotted-abbreviation ports, plus the owner-added
`DANISH_TYPOS` Wikipedia typo-list rule (#14) pinned at an exact-count only-Rust
allowance of 0 corpus matches and 16 owner-added DanNet-derived
confusable-word rulegroups (#15) that the corpus does not trigger; the M4
refresh moved the tagger dict to the current Stavekontrolden 2.9.137 data and
added five owner-authored disambiguation rulegroups (#18) — the dict refresh
changes 2 corpus lines (same corrections, current-upstream readings the stale
Java dict lacks) pinned exactly as `Ordgentagelse`/`unde` 1+1 only-Java/
only-Rust, the new rulegroups change none); sv is at 0/0/0 (#11,
resolved by the suggestion-engine port, plus the owner-added
SALDO-derived confusable-word and coherency pairs (#16) and the owner-added
default-off `VECKODAG_DATUM` `<filter>` rules — the sv module's first filter
class `org.languagetool.rules.sv.DateCheckFilter` (#17) — that the corpus
does not trigger; the B1 lexicon swap rebuilt both Swedish dictionaries from
the Språkbanken SALDO morphological lexicon (CC BY 4.0, `data/sv/README.md`)
onto the unchanged SUC-style tagset and keeps the golden at 0/0/0 with no
allowance); is is at 0/0/0 (#12); eo is at 0/0/0
(#10, resolved by the wrong-split and `twowords` ports); ast is at 0/0/0 (#13); br is at 0/0/0 (#14); tl is at 0/0/0 (#15, resolved: the morfologik speller `getFrequency` now
reproduces Java's signed-byte frequency arithmetic, so the
frequency-weighted `MORFOLOGIK_RULE_TL` suggestions order identically); crh is at 0/0/0 (#16, resolved: Java's `UNICODE_CASE` folds `ı`/`İ` into the ASCII `i`/`I` class and `lt_pattern` now adds them); be is at 0/0/0 (#17); ru is at 0/0/0 (#18); uk is at 0/0/0 (#19, resolved); sr is at 0/0/0 (#20; the golden is captured from a forward-ported in-container copy of the reactor-excluded `sr` module, `scripts/oracle/sr/check-diff-sr.sh`, D-285/D-287/D-288). `no`, `nrd`, `nn` and `gn` are hand-authored

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fiduswriter/LingoTweaker](https://github.com/fiduswriter/LingoTweaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->

---
trigger: always_on
description: This file is the working guide for an AI agent about to modify this codebase. Read it before touching anything.
---

# NattyDate — Codebase Guide for AI Agents

This file is the working guide for an AI agent about to modify this codebase. Read it before touching anything.

---

## What This Project Does

NattyDate is a deterministic, fuzzy-scoring natural language date/time preprocessor. Given messy human strings like `"nxt fri 14:00"` or `"last monday at noon"`, it returns normalized structures, formatted timestamps, or a JSON token AST. No ML, no network calls, no system clock in tests.

---

## Key Files

| File | Role |
|---|---|
| `src/lib.rs` | Everything: pipeline, scoring, resolution, formatting |
| `src/main.rs` | CLI wrapper and test runner — thin; no business logic |
| `tests.json` | 104-case test suite with fixed `mock_now: 2026-03-18` |
| `SKILL.md` | User-facing quick reference (how to use the tool) |
| `CONTRIBUTING.md` | How to add words, holidays, modifiers |

---

## Build and Test

```bash
cargo build                          # debug build
cargo build --release                # optimised binary
cargo test                           # runs all 104 cases via #[test] in lib.rs
cargo run -- test --verbose          # same suite via CLI runner
cargo run -- "tomorrow at 3pm" -f "YYYY-MM-DD HH:mm:ss"
cargo run -- "nxt fri 14:00" -o json
cargo run -- test --test-file custom.json
```

All tests are deterministic: `mock_now = 2026-03-18` (a Wednesday) is injected via `ParseConfig.mock_now`. Never rely on `Local::now()` in tests.

---

## Pipeline Architecture

```
Input string
    │
    ▼
normalize()          — lowercase, phrase substitution, whitespace collapse
    │
    ▼
tokenize()           — split into RawToken variants: Word / Number / DateNumeric / Time / TimeZone
  ├─ convert_compound_numbers()   "nine thirty" → "9 30"
  └─ resolve_time_phrases()       "half past 10" → "10:30"
    │
    ▼
tokenize_and_classify()          — for each raw token, call evaluate_token()
  ├─ evaluate_token()             returns Vec<ScoredToken>, sorted by score desc
  │   ├─ parse_time_scored()
  │   ├─ parse_date_numeric_scored()
  │   ├─ parse_timezone_scored()
  │   ├─ i32 parse
  │   ├─ HashMap exact dict lookup   ← O(1), short-circuits fuzzy
  │   └─ Levenshtein fuzzy scan      ← only runs on dict miss
  └─ recombine adjacent tokens if combined score > individual avg + RECOMBINE_GAIN
    │
    ▼
apply_context_boosts()           — 4-phase, accumulate then apply
  Phase 1: collect score boosts (temporal adjacency, modifier→weekday, num+unit)
  Phase 2: apply boosts, sort Unknown candidate lists once each
  Phase 3: At + Number → Time structural conversion
  Phase 4: promote Unknown ≥ THRESHOLD_PROMOTE to Known
    │
    ▼
resolve()            — consumes Modifier/Weekday/Holiday/RelativeDay into DateNumeric
  preprocess_ago_patterns()      — converts "N unit ago" triples to DateNumeric first
    │
    ▼
to_canonical() / format_custom() — produce output string
```

---

## Token Types

```rust
// After resolve(), you will mostly see:
Token::Known(ScoredToken { token: KnownToken::DateNumeric { y, m, d }, score })
Token::Known(ScoredToken { token: KnownToken::Time { hour, min, sec, formatted }, score })
Token::Known(ScoredToken { token: KnownToken::TimeZone(TimeZoneKind::Named("EDT")), score })
Token::Noise(String)           // filtered out before output
Token::Unknown { word, candidates }  // score < threshold, passed through
```

`Modifier`, `Weekday`, `RelativeDay`, and `Holiday` tokens are consumed by `resolve()` and do not appear in final output — they are converted to `DateNumeric`. The JSON `-o json` output reflects the resolved state.

---

## Scoring Constants (top of lib.rs)

```rust
SCORE_EXACT         = 1.0   // dictionary exact match
SCORE_DICT_TIME     = 0.9   // exact match for named time words (morning, noon…)
SCORE_EXPLICIT      = 0.95  // unambiguous: ISO, HH:MM, am/pm
SCORE_NUMERIC       = 0.9   // plain integer
SCORE_DATE_YMD      = 0.95  // three-part date with year, unambiguous
SCORE_DATE_YMD_AMBIG= 0.9   // three-part date with year, ambiguous m/d order
SCORE_DATE_MD       = 0.75  // two-part date (no year)
SCORE_DATE_AMBIG    = 0.7   // two-part date, both parts ≤ 12
SCORE_TIME_COLON    = 0.95  // HH:MM with colon
SCORE_TIME_DEFAULT  = 0.8   // other separators, no am/pm
SCORE_TIME_SINGLE   = 0.7   // bare hour with no suffix

FUZZY_BONUS_PREFIX  = 0.1   // candidate word starts with the query
FUZZY_BONUS_FIRST_CHAR = 0.05
FUZZY_MAX_SCORE     = 0.95  // cap so fuzzy never equals exact
FUZZY_MIN_ACCEPT    = 0.65  // minimum to surface a fuzzy candidate
FUZZY_MAX_LEN_DIFF  = 2     // skip pairs whose lengths differ by more

BOOST_TEMPORAL_ADJ  = 0.1   // Time adjacent to Date/TZ
BOOST_MODIFIER_CTX  = 0.2   // Weekday candidate following a Modifier
BOOST_RELATIVE_TIME = 0.15  // RelativeDay candidate adjacent to Time
BOOST_NUM_UNIT_PAIR = 0.2   // Number directly before a Unit

THRESHOLD_PROMOTE   = 0.65  // Unknown promoted to Known
THRESHOLD_KNOWN     = 0.65  // Single-candidate accepted as Known
THRESHOLD_MARGIN    = 0.09  // Score gap required for top candidate to win
RECOMBINE_MIN       = 0.8   // Min score for recombined token to win
RECOMBINE_GAIN      = 0.15  // Combined must beat per-token avg by this
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/troy74) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->

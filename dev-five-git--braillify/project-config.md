---
trigger: always_on
description: Korean + Math Braille encoding engine implementing 2024 Korean Braille Standard.
---

# CORE LIBRARY (libs/braillify)

Korean + Math Braille encoding engine implementing 2024 Korean Braille Standard.

## STRUCTURE

```
src/
├── lib.rs              # Main encode() entry + encode_to_unicode() / encode_to_braille_font()
├── cli.rs              # CLI: REPL + one-shot mode (feature-gated)
├── main.rs             # Binary entry point
├── encoder.rs          # DocumentIR construction, token + char engine orchestration
├── char_struct.rs      # CharType enum (Korean/English/Number/Symbol/MathSymbol/Fraction)
├── korean_char.rs      # Full Korean syllable encoding
├── korean_part.rs      # Standalone jamo (consonant/vowel) encoding
├── jauem/              # Consonant handling
│   ├── choseong.rs     # Initial consonants
│   └── jongseong.rs    # Final consonants
├── moeum/              # Vowel handling
│   └── jungsong.rs     # Medial vowels
├── english.rs          # English letter encoding
├── english_logic.rs    # English context detection
├── number.rs           # Number encoding
├── fraction.rs         # Fraction handling (Unicode + LaTeX)
├── math_symbol_shortcut.rs  # PHF math symbol lookup table
├── symbol_shortcut.rs       # PHF general symbol lookup table
├── word_shortcut.rs         # PHF word abbreviation lookup table
├── unicode.rs          # Internal braille code ↔ Unicode Braille conversion
├── split.rs            # Korean jamo decomposition
├── utils.rs            # Helper functions
└── rules/              # Rule engine (see below)
```

## ENCODING PIPELINE

```
Input text
  ↓ DocumentIR::parse()         (tokenize into Word/Space/Mode tokens)
  ↓ TokenRuleEngine::apply_all() (token-level rules by phase)
  │   ├── LatexMergeRule         (merge $...$ across spaces)
  │   ├── LatexFractionRule      (detect $\frac{}{})$)
  │   ├── LatexMathRule          (strip LaTeX → math notation)
  │   ├── InlineFractionRule     (detect N/N inline fractions)
  │   ├── MathExpressionTokenRule (detect & encode math expressions)
  │   └── ...other token rules
  ↓ emit()                      (character-level encoding)
      ├── Token::Word → RuleEngine (BrailleRule trait, char-by-char)
      ├── Token::Space → braille space byte
      ├── Token::Fraction → fraction encoding
      └── Token::PreEncoded → pass-through (from math encoder)
```

## RULE ARCHITECTURE

### Two parallel rule systems

| System | Trait | Engine | Operates On | Used By |
|--------|-------|--------|-------------|---------|
| Korean (char-level) | `BrailleRule` | `RuleEngine` | Individual characters (`CharType`) | Korean text encoding |
| Math (token-level) | `MathTokenRule` | `MathTokenEngine` | Token sequences (`MathToken`) | Math expression encoding |

### BrailleRule (Korean, character-level)

```rust
trait BrailleRule: Send + Sync {
    fn meta(&self) -> &'static RuleMeta;
    fn phase(&self) -> Phase;           // Preprocessing → CoreEncoding → InterCharacter
    fn matches(&self, ctx: &RuleContext) -> bool;
    fn apply(&self, ctx: &mut RuleContext) -> Result<RuleResult, String>;
}
```

Registered in `encoder.rs` → processes one character at a time via `RuleContext`.

### MathTokenRule (Math, token-level)

```rust
trait MathTokenRule: Send + Sync {
    fn name(&self) -> &'static str;
    fn priority(&self) -> u16;          // Lower = runs first (10=lookahead, 50=core, 100=symbol)
    fn matches(&self, tokens: &[MathToken], index: usize, state: &MathEncodeState) -> bool;
    fn apply(&self, tokens: &[MathToken], index: usize, result: &mut Vec<u8>,
             state: &mut MathEncodeState, engine: &MathTokenEngine) -> Result<MathTokenResult, String>;
}
```

Registered in `encoder.rs::build_math_engine()` → processes parsed MathToken sequences with lookahead.

### Math rule structs (in respective rule files)

| Priority | Struct | File | Handles |
|----------|--------|------|---------|
| 10 | `FractionReversalRule` | rule_7.rs | Denominator-first simple fractions |
| 10 | `ConditionalProbFractionRule` | rule_7.rs | =a/b with \| pattern |
| 10 | `CombinatoricsRule` | rule_12.rs | nPr, nCr |
| 50 | `NumberRule` | rule_1.rs | Number tokens |
| 50 | `VariableRule` | rule_12.rs | Lowercase variables |
| 50 | `UpperVariableRule` | rule_12.rs | Uppercase variables |
| 50 | `OperatorRule` | rule_2.rs | Arithmetic operators |
| 50 | `FunctionNameRule` | rule_47.rs | log, lim, sin, cos... |
| 50 | `BracketRule` | rule_6.rs | Open/close parentheses |
| 50 | `SuperscriptRule` | rule_18.rs | Superscript content |
| 50 | `SubscriptRule` | rule_19.rs | Subscript content |
| 50 | `DecimalPointRule` | rule_8.rs | Decimal points |
| 50 | `PrimeRule` | rule_53.rs | Prime marks |
| 100 | `MathSymbolRule` | encoder.rs | All math symbols (30+ dispatch chain) |

## KEY TYPES

| Type | Location | Purpose |
|------|----------|---------|
| `CharType` | `char_struct.rs` | Input character classification |
| `BrailleRule` | `rules/traits.rs` | Korean char-level rule trait |
| `MathTokenRule` | `rules/math/math_token_rule.rs` | Math token-level rule trait |
| `MathTokenEngine` | `rules/math/math_token_rule.rs` | Math rule dispatch engine |
| `MathToken` | `rules/math/parser.rs` | Parsed math expression token |
| `MathEncodeState` | `rules/math/math_token_rule.rs` | Shared math encoding state |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dev-five-git/braillify](https://github.com/dev-five-git/braillify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

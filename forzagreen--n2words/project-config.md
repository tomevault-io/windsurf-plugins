---
trigger: always_on
description: n2words: Number to words converter. ESM + UMD, Node >=22, zero dependencies.
---

# CLAUDE.md

n2words: Number to words converter. ESM + UMD, Node >=22, zero dependencies.

## Quick Reference

- **Language codes**: IETF BCP 47 (`en-US`, `zh-Hans-CN`, `fr-BE`)
- **Imports**: `import { toCardinal, toOrdinal, toCurrency } from 'n2words/en-US'`
- **Forms**: Cardinal (`toCardinal`), Ordinal (`toOrdinal`), Currency (`toCurrency`)

## Project Structure

```text
src/
├── {lang-code}.js       # One file per language (70+)
└── utils/
    ├── parse-cardinal.js    # Cardinal form parsing (decimals, negatives)
    ├── parse-ordinal.js     # Ordinal form parsing (positive integers only)
    ├── parse-currency.js    # Currency form parsing (dollars, cents)
    ├── scale.js             # Pure *Max producers (western/myriad/indian/longScale/bounded/UNBOUNDED)
    ├── check-max.js         # checkMax: throws RangeError past a form's declared ceiling
    ├── resolve-options.js   # resolveOptions: applies a form's exported defaults, validates options
    ├── expand-scientific.js # Scientific notation expansion
    └── is-plain-object.js   # Object type checking
```

## Language File Pattern

A language exports one, two, or all three forms — `toCardinal`, `toOrdinal`, `toCurrency`
(at least one; forms are added incrementally, so export only what you implement). Each form
it exports must uphold the **conversion contract** (enforced by `test/contract.test.js`): for *any* input,
return a well-formed string **or** throw `RangeError` — never malformed output. Don't invent
vocabulary past your largest scale word: each form declares a bigint ceiling
(`cardinalMax`/`ordinalMax`/`currencyMax`, or `UNBOUNDED`), derived from your own scale table
via a `scale.js` helper so it can't drift, and guards the entry point with `checkMax`
(O(1), before building). Full spec in `docs/range-contract.md`; the gate
(`test/range-contract.test.js`) verifies every declared ceiling.

```javascript
import { parseCardinalValue } from './utils/parse-cardinal.js'
import { parseCurrencyValue } from './utils/parse-currency.js'
import { parseOrdinalValue } from './utils/parse-ordinal.js'
import { checkMax } from './utils/check-max.js'
import { western } from './utils/scale.js' // pick the helper matching your grouping

// Each form's ceiling, derived from your own table. western(n) treats n as the
// count of scale words above units — pass SCALES.length when the table starts at
// "thousand", or SCALES.length - 1 when index 0 is an empty units slot (see scale.js).
export const cardinalMax = western(SCALES.length) // smallest value the form refuses
export const ordinalMax = western(SCALES.length)  // often lower — derive separately if so
export const currencyMax = western(SCALES.length) // usually shares the cardinal ceiling

function toCardinal (value) {
  const { isNegative, integerPart, decimalPart } = parseCardinalValue(value)
  // Pass decimalPart only when the fraction routes through the scale builder
  // (omit it for digit-by-digit languages, which have no decimal ceiling).
  checkMax(integerPart, cardinalMax, decimalPart)
  // integerPart is bigint, handle isNegative prefix and decimalPart suffix
}

function toOrdinal (value) {
  const integerPart = parseOrdinalValue(value)
  checkMax(integerPart, ordinalMax)
  // positive integers only
}

function toCurrency (value) {
  const { isNegative, dollars, cents } = parseCurrencyValue(value)
  checkMax(dollars, currencyMax) // cents are ≤ 99, safe
  // dollars/cents are bigints
}

export { toCardinal, toOrdinal, toCurrency }
```

Beware **silently-wrong** builders: if yours drops the scale word past its table (e.g.
`if (SCALES[i - 1])`, `if (!meta) return …`) it returns well-formed-but-wrong output that
fuzzing can't catch — derive the ceiling by reading the table, not by probing for garbage.

Language files are **self-contained**: duplicate small helpers rather than share them.
Extract a util only for the API contract (parsing, options, the range guard) or
genuinely universal single-purpose logic.

## Options Pattern

A form that accepts options declares an **options contract** — enforced by
`test/options-contract.test.js`: any form whose function takes an options
parameter **must** export its `<form>Defaults` (a form without one fails CI).

```javascript
import { resolveOptions } from './utils/resolve-options.js'

/**
 * @typedef {object} CardinalOptions
 * @property {('masculine'|'feminine')} [gender] - Grammatical gender of the number
 */

/** @type {Required<CardinalOptions>} */
export const cardinalDefaults = { gender: 'masculine' }

/** @type {{ gender: ReadonlyArray<Required<CardinalOptions>['gender']> }} */
export const cardinalValues = { gender: ['masculine', 'feminine'] } // enum options only

/**
 * @param {number | string | bigint} value
 * @param {CardinalOptions} [options]
 * @returns {string}
 */
function toCardinal (value, options) {
  const { isNegative, integerPart, decimalPart } = parseCardinalValue(value)
  const { gender } = resolveOptions(options, cardinalDefaults, cardinalValues)
  // Pass explicit values to internal functions, not options object
}
```

Each fact has exactly one home, and machines hold every seam:

- **type + description** → the `@typedef` (one `@property` per option — lint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forzagreen/n2words](https://github.com/forzagreen/n2words) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->

---
trigger: always_on
description: `country-codes-list` is an npm package with one record per country (250 records): ISO codes, currency, official language, calling code, phone number lengths, tax identifier, region and flag. It has zero runtime dependencies and ships TypeScript types.
---

# country-codes-list: guide for AI coding agents

`country-codes-list` is an npm package with one record per country (250 records): ISO codes, currency, official language, calling code, phone number lengths, tax identifier, region and flag. It has zero runtime dependencies and ships TypeScript types.

This file serves two readers: an agent that USES the package in a consumer project (read up to "Pair with"), and an agent that CONTRIBUTES to this repo (read "Contributor workflow").

```bash
npm install country-codes-list
```

```js
const countryCodes = require("country-codes-list");   // CommonJS
import * as countryCodes from "country-codes-list";    // through Node's CJS interop, no native ESM build
```

## API cheat sheet

| Export | Signature | Example |
| --- | --- | --- |
| `all()` | `(): CountryData[]` | `all().length // 250` |
| `filter(key, value)` | `(key: CountryScalarProperty, value: string): CountryData[]` | `filter("currencyCode", "XCG").map(c => c.countryCode) // ['CW', 'SX']` |
| `findOne(key, value)` | `(key: CountryScalarProperty, value: string): CountryData \| undefined` | `findOne("countryCodeAlpha3", "ARG").countryNameEn // 'Argentina'` |
| `findOneByCode(code)` | `(code: string): CountryData \| undefined` | `findOneByCode("UK").countryCode // 'GB'`, `findOneByCode("840").countryCode // 'US'` |
| `customList(key?, label?, opts?)` | `(key = "countryCode", label = "{countryNameEn} ({countryCode})", { filter? }): Record<string, string>` | `customList("countryCode", "{flag} {countryNameEn}")["US"] // '🇺🇸 United States of America'` |
| `customGroupedList(key?, label?, opts?)` | `(key = "countryCallingCode", label = same, { filter? }): Partial<Record<string, string[]>>` | `customGroupedList("countryCallingCode", "{countryCode}")["1"] // ['AG', 'AI', ..., 'UM'] (26)` |
| `customArray(fields?, opts?)` | `<F>(fields = { name, value }, { sortBy?: keyof F, sortDataBy?: CountryScalarProperty, filter? }): Record<keyof F, string>[]` | `customArray({ name: "{countryNameEn}", value: "{countryCode}" }, { sortBy: "name" })` |
| `utils.groupBy(array, key)` | `<T>(array: T[], key: keyof T): Record<string, T[]>` | `utils.groupBy(all(), "region")["Arab States"].length // 22` |

Types: `CountryData` (one record), `CountryProperty` (every key), `CountryScalarProperty` (string-valued keys only). All key parameters above take `CountryScalarProperty`.

Templates use `{placeholder}` syntax with any string or number field. Array fields and unknown names stay in the output as written.

## Field table

| Field | Type | Populated | Gotcha |
| --- | --- | --- | --- |
| `countryNameEn` | `string` | all | |
| `countryNameLocal` | `string` | all | |
| `countryCode` | `string` | all | ISO 3166-1 alpha-2. Unique. Never `UK` or `EL`. |
| `countryCodeAlpha3` | `string` | all | ISO 3166-1 alpha-3. Unique. |
| `countryCodeNumeric` | `string` | all but `XK` | Three digits with leading zeros (`"004"`). Generated. |
| `altCodes` | `string[]` optional | `GB` (`["UK"]`), `GR` (`["EL"]`) | Absent on other records. Search it with `findOneByCode`. |
| `currencyCode` | `string` | all but `AQ` | ISO 4217. |
| `currencyNameEn` | `string` | all but `AQ` | |
| `currencyNumeric` | `string` | all but `AQ` | Three digits. Generated. |
| `currencyDecimals` | `number \| null` | all but `AQ` (`null`) | `0` for JPY, `3` for BHD. Generated. |
| `currencySymbol` | `string` | all but `AQ` | Not unique: 29 currencies show `$`. Falls back to the code (`CHF`). Generated. |
| `tinType` | `string` | 62 of 250 | Empty string means "not recorded". |
| `tinName` | `string` | 64 of 250 | Empty string means "not recorded". |
| `officialLanguageCode` | `string` | all | ISO 639-1, or ISO 639-3 when no 639-1 code exists. First official language only. |
| `officialLanguageNameEn` | `string` | all | |
| `officialLanguageNameLocal` | `string` | all | |
| `countryCallingCode` | `string` | all | E.164 country code only. Digits, no `+`, no area code. Shared by many countries (`1`, `44`, `61`). |
| `areaCodes` | `string[]` | NANP members except `US` and `UM`, plus `CC`, `CX`, `SJ` | Empty means "not recorded", not "none". |
| `nationalNumberLengths` | `number[]` | all but `AQ`, `BV`, `GS`, `HM`, `PN`, `TF`, `UM` | A set, not a range. Fixed-line and mobile only. Generated. |
| `region` | `string` | all | Six values adapted from ITU: Africa, Arab States, Asia & Pacific, Europe, North America, South/Latin America. |
| `flag` | `string` | all | Emoji derived from `countryCode`. |

## Rules for correct use

- Use `findOneByCode` for codes from outside your code (locales, VAT numbers, APIs). It accepts alpha-2, alpha-3, `altCodes` and 3-digit numeric codes, case-insensitive.
- Use `findOne("countryCode", x)` only for exact, uppercase ISO codes. `findOne("countryCode", "UK")` is `undefined`.
- Use `includes`, never min/max, on `nationalNumberLengths`. NL is `[9, 11]`, so 10 digits is invalid.
- Strip the trunk prefix (the leading `0`) before you compare a length. Keep the area code.
- Treat an empty `areaCodes` or `nationalNumberLengths` as "not recorded", never as "none".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Synergy-Shock/country-codes-list](https://github.com/Synergy-Shock/country-codes-list) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->

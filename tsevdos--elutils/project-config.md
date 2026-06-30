---
trigger: always_on
description: elUtils (`@tsevdos/el-utils`) is a zero-dependency TypeScript utility library providing convenient functions for Greek-specific business operations. The library focuses on geolocation, validation, date handling, formatting, and language utilities for Greek applications.
---

# GitHub Copilot Instructions for elUtils

## Project Overview

elUtils (`@tsevdos/el-utils`) is a zero-dependency TypeScript utility library providing convenient functions for Greek-specific business operations. The library focuses on geolocation, validation, date handling, formatting, and language utilities for Greek applications.

## Core Principles

1. **Zero Dependencies**: Do not suggest or add any external dependencies
2. **Type Safety**: All code must be strongly-typed using TypeScript
3. **Pure Functions**: Functions should be pure, with no side effects
4. **Immutability**: Never mutate input parameters or data structures
5. **Bilingual Support**: Support both Greek (`el`) and English (`en`) locales where applicable
6. **Comprehensive Testing**: Every function must have corresponding Vitest tests

## Project Structure

```
src/
├── dateUtils.ts         # Date-related utilities (days, months, holidays, etc.)
├── formatUtils.ts       # Formatting utilities (weights, etc.)
├── geoUtils.ts          # Geographic/geolocation utilities (cities, regions, postal codes)
├── languageUtils.ts     # Greek language utilities (text comparison, normalization)
├── validationUtils.ts   # Validation utilities (VAT, AMKA, postal codes, phones)
├── types.ts             # TypeScript type definitions
└── __tests__/           # Jest test files
```

## Coding Standards

### Function Structure

1. **JSDoc Comments**: Every exported function must have JSDoc documentation with:
   - Clear description
   - `@param` tags for all parameters with types and descriptions
   - `@returns` tag describing the return value
   - `@example` tags when helpful
   - Links to external references when applicable (e.g., validation algorithms)

Example:

```typescript
/**
 * Validates the given postal code.
 *
 * @param {string} postalCode - The postal code to validate.
 *
 * @returns {boolean} `true` if the postal code is valid, `false` otherwise.
 */
export function validatePostalCode(postalCode: string): boolean {
  // implementation
}
```

2. **Options Pattern**: For functions with multiple parameters, use an options object with defaults:

```typescript
type FunctionOptions = {
  locale?: "el" | "en";
  format?: "full" | "short";
};

export function myFunction({ locale = "el", format = "full" }: FunctionOptions = {}) {
  // implementation
}
```

3. **Type Definitions**: Define types in `types.ts` and import them. Use descriptive type names.

4. **Data Imports**: Import JSON data from the `data/` directory at the top of files:

```typescript
import dataEl from "../data/data-el.json";
import dataEn from "../data/data-en.json";
```

### Naming Conventions

- **Functions**: Use camelCase with descriptive verbs (e.g., `validatePostalCode`, `getAdministrativeRegions`)
- **Types**: Use PascalCase (e.g., `Region`, `TaxOffice`, `City`)
- **Constants**: Use UPPER_SNAKE_CASE for constants (e.g., `MOUNT_ATHOS_REGION_ID`)
- **Locale Objects**: Use pattern `{ el: dataEl, en: dataEn } as const`

### Testing Standards

1. **Test File Location**: Place tests in `src/__tests__/` with pattern `[module].test.ts`
2. **Test Framework**: Use Vitest for all tests
3. **Test Structure**: Use `describe` blocks for each function and `it` blocks for specific cases
4. **Test Coverage**: Cover:
   - Valid inputs (multiple cases)
   - Invalid inputs (multiple edge cases)
   - Boundary conditions
   - Different locales when applicable
   - Different formats/options when applicable

Example:

```typescript
describe("validatePostalCode", () => {
  it("returns true on existing postal codes", () => {
    expect(validatePostalCode("17562")).toBe(true);
    expect(validatePostalCode("30005")).toBe(true);
  });

  it("returns false on not existing postal codes", () => {
    expect(validatePostalCode("12345")).toBe(false);
    expect(validatePostalCode("11111")).toBe(false);
  });
});
```

## Module-Specific Guidelines

### dateUtils.ts

- Return arrays of strings in the specified locale and format
- Support formats: `full`, `short`, `min` (and `alternative` for months)
- Default locale is `el`, default format is `full`
- Use the JSON data from `data/dates.json`
- Exported functions: `getDays`, `getMonths`, `getQuarters`, `getEras`, `getHolidays`

### formatUtils.ts

- Provide bilingual formatting for various units
- Support `full`, `full_single`, `short` formats
- Include international symbols when requested
- Use data from `data/weights.json`
- Exported functions: `formatWeight`

### geoUtils.ts

- Handle Greek administrative divisions: regions, units, municipalities, prefectures
- Special handling for Mount Athos (region ID 14, prefecture ID 55)
- Provide search functionality with Greek text normalization
- Support postal codes, cities, tax offices, and countries
- Always use the `convertsGreekTextToComparableUpperCase` helper for Greek text comparisons

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tsevdos/elUtils](https://github.com/tsevdos/elUtils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->

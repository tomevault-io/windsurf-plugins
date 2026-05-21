---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js web application that helps Slovenian residents generate XML tax forms for Revolut Savings Accounts. The application processes Revolut CSV statements client-side and generates XML files for submission to eDavki (Slovenian tax authority).

**Key constraint**: All data processing happens client-side in the browser for privacy. No backend server or external API calls (except for static conversion-rates.json).

**Recent major architectural change (Jan 2026)**: Implemented FIFO (First-In-First-Out) matching logic to properly match BUY and SELL orders for accurate cost basis reporting. This replaces the previous simple order listing approach and ensures tax compliance.

## Development Commands

```bash
# Install dependencies
yarn install

# Start development server (localhost:3000)
yarn dev

# Build for production
yarn build

# Start production server
yarn start

# Run linter
yarn lint

# Run tests (Jest)
yarn test

# Run tests in watch mode
yarn test:watch

# Update currency conversion rates (requires Python)
python scripts/update-conversion-rates.py
```

## Core Architecture

### Data Flow Pipeline

The application follows a linear processing pipeline:

1. **User Input** → CSV file + 8-digit tax number + tax year
2. **CSV Validation** (`lib/revolut-parser.ts`) → Validates file format before processing
3. **CSV Parsing** (`lib/revolut-parser.ts`) → Extracts transactions by currency/ISIN
4. **Currency Conversion** → Uses `public/conversion-rates.json` to convert to EUR
5. **FIFO Matching** (`lib/cost-basis.ts`) → Matches BUY/SELL orders chronologically for tax reporting
6. **History Validation** → Ensures all SELLs can be matched with BUYs (complete transaction history)
7. **XML Generation** (`lib/tax-generator.ts`) → Creates eDavki-compliant XML files using matched transactions
8. **Output** → Downloadable XML files + human-readable report

### Directory Structure

- **`/app`** - Next.js App Router pages (layout, main page, globals.css)
- **`/components`** - React components (UI layer)
    - `file-upload.tsx` - Core component handling upload, processing, and results (465 lines)
    - `eligibility-check.tsx` - Pre-qualification form
    - `instructions-accordion.tsx` - User guide
    - `/ui` - shadcn/ui component library (52 reusable components)
- **`/lib`** - Business logic (parsing, generation, utilities)
    - `revolut-parser.ts` - CSV parsing, validation, and currency conversion
    - `cost-basis.ts` - FIFO matching logic for BUY/SELL orders (276 lines)
    - `tax-generator.ts` - XML generation for eDavki forms using FIFO-matched data
    - `report-generator.ts` - Human-readable summary reports
    - `*.test.ts` - Jest test files (cost-basis, tax-generator)
- **`/hooks`** - Custom React hooks (toast notifications)
- **`/public`** - Static assets including `conversion-rates.json`
- **`/scripts`** - Utility scripts
    - `update-conversion-rates.py` - Python script to update currency conversion rates

### Key Technical Details

**CSV Validation** (`lib/revolut-parser.ts`):

- `validateRevolutCSV()` validates file before processing
- Checks for minimum content length (at least 3 rows)
- Looks for Revolut-specific headers: "Summary", "Transactions for", "Flexible Cash Funds"
- Returns `ValidationResult` with error messages in Slovenian

**CSV Parsing Strategy** (`lib/revolut-parser.ts`):

- Detects fund sections by currency using header regex: `- ([A-Z]{3})`
- Switches to "transaction mode" when line starts with "Transactions for"
- Extracts ISIN codes via regex: `\b[A-Z]{2}[0-9A-Z]{9}[0-9]\b`
- Processes three transaction types: BUY, SELL, and Interest PAID
- Converts all amounts to EUR using historical rates from conversion-rates.json

**FIFO Matching Logic** (`lib/cost-basis.ts`):

- **Core Algorithm**: `matchTransactionsFIFO()` implements First-In-First-Out matching
    - Sorts all transactions chronologically
    - Maintains a queue of `BuyLot[]` (oldest first)
    - BUY orders add new lots to the queue
    - SELL orders consume from the front of the queue (oldest first)
    - Returns `MatchingResult` with matches grouped by tax year and remaining inventory

- **History Validation**: `validateHistory()` ensures complete transaction history
    - Tracks running inventory for each fund
    - Detects if any SELL happens before sufficient BUY orders
    - Returns detailed deficit information in Slovenian

- **Helper Functions**:
    - `getMatchesForYear()` - extracts matches for a specific tax year
    - `getConsumedBuysForYear()` - gets unique BUY orders consumed by SELLs in a tax year
    - `calculateTaxYearSummary()` - calculates totals for reporting

- **Key Data Structures**:
    - `BuyLot` - tracks remaining quantity from each BUY order
    - `MatchedSell` - links each SELL to the BUY orders that cover it
    - `MatchingResult` - contains all matches grouped by year plus remaining inventory

**XML Generation** (`lib/tax-generator.ts`):

- **Uses FIFO-matched data** from `cost-basis.ts` instead of raw order lists
- **Doh_KDVP**: Capital gains form for BUY/SELL orders

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matevzpoljanc/revolut-savings-account-tax-reports](https://github.com/matevzpoljanc/revolut-savings-account-tax-reports) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->

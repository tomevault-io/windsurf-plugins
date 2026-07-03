---
trigger: always_on
description: Personal budget tracker for tracking monthly expenses, bank balances, and calculating available spending money. Internet-facing with passkey authentication (no user management).
---

# SSBudget - Claude Context File

## Project Overview

Personal budget tracker for tracking monthly expenses, bank balances, and calculating available spending money. Internet-facing with passkey authentication (no user management).

## UI Design Principles

**Spreadsheet-like efficiency** - The UI should feel like a well-designed spreadsheet:
- **Concise**: Maximum information density, minimal chrome
- **Direct manipulation**: Edit in place, no unnecessary modals or multi-step wizards. But explicit submission is ok when needed
- **Minimal clicks**: Common actions (update balance, mark paid) should be 1-2 clicks
- **Scannable**: Numbers aligned, status visible at a glance

Think "Google Sheets for personal budget" not "enterprise dashboard with cards everywhere".

## Core Concepts

### Period
- Starts when paycheck arrives (~25th of month, flexible)
- Ends when next paycheck arrives
- All calculations are relative to current period

### Expense Types

1. **Planned Expenses** - Fixed monthly bills (rent, subscriptions, etc.)
   - Have an estimated amount
   - Get marked as "paid" with actual amount
   - Unpaid ones contribute their estimate to predicted expenses
   - Estimate can be: fixed value, last month's actual, or historical average

2. **Estimated Expenses** - Variable ongoing costs (groceries, fuel, etc.)
   - Have a monthly estimate
   - Never explicitly "paid" - consumed implicitly over time
   - Scale with remaining period (10 days left = 1/3 of monthly estimate)
   - Can toggle whether included in remaining balance calculation

### Savings

**Savings Accounts** - Buckets for accumulating money (emergency fund, vacation, etc.)
- Have a currency and current balance
- Balance is editable directly (for corrections/initial setup)
- Can have an optional monthly target (planned savings amount)
- Transactions (inflows/outflows) modify the balance

**Savings Transactions** - Individual money movements
- Positive = inflow (contributing to savings)
- Negative = outflow (withdrawing from savings)
- Can have multiple per period (e.g., +500, -100, -200)
- Optional note for context

### Key Calculation
```
Free Money = Total Balance - Predicted Expenses - Remaining Savings
Daily Budget = Free Money / Days Until Period End
```

Where:
- `Predicted Expenses = Sum(unpaid planned estimates) + Scaled(estimated expenses)`
- `Remaining Savings = Sum(plannedMonthly - period contributions) for accounts with targets`
- Period contributions = sum of transactions for current period per savings account

## Tech Stack

| Layer       | Technology                               |
|-------------|------------------------------------------|
| Language    | Scala 3.5.2                              |
| Backend     | cats-effect, tapir, http4s               |
| Frontend    | Laminar (Scala.js SPA)                   |
| API         | tapir (shared endpoint definitions)      |
| Database    | SQLite + Flyway migrations               |
| JSON        | circe                                    |
| CSS         | Bootstrap 5 (CSS-only)                   |
| Bundler     | Vite + vite-plugin-scalajs               |
| Auth        | Passkeys (WebAuthn) via java-webauthn-server |
| Deployment  | Docker + fly.io                          |

## Reference Projects

- **workflow4s-web-ui** (`/Users/krever/Projects/priv/workflow4s/workflows4s-web-ui`) - Reference for Vite + Scala.js setup
- **laminar-full-stack-demo** (https://github.com/raquo/laminar-full-stack-demo) - Reference for Laminar full-stack architecture
- **forms4s** (`/Users/krever/Projects/priv/forms4s`) - Form/datatable library to extend with Laminar support
- **business4s ecosystem** (https://business4s.org/) - Parent OSS ecosystem

### forms4s Integration Strategy
1. Use `forms4s-core` for table/form state management (no UI dependency)
2. Build `forms4s-laminar` module as part of this project (can be extracted later)
3. Leverage existing: TableDef, TableState, filtering, sorting, pagination, URL state encoding

## Data Model (Conceptual)

```
ExpenseDefinition:
  - id, name, type (planned|estimated)
  - estimateMode (fixed|lastMonth|average)
  - fixedEstimate (optional)
  - includeInBalance (for estimated type)

Period:
  - id, startDate, endDate (nullable until closed)

ExpenseRecord (for planned expenses):
  - periodId, expenseDefId, paidAmount (nullable), paidDate

BalanceSnapshot:
  - accountId, amount, currency, timestamp

Account:
  - id, name, currency (PLN|EUR)

SavingsAccount:
  - id, name, currency (PLN|EUR)
  - currentBalance (cents, editable directly)
  - plannedMonthly (optional target per period)

SavingsTransaction:
  - id, accountId, periodId
  - amount (positive = inflow/saving, negative = outflow/withdrawal)
  - note (optional)
  - createdAt

ExchangeRate:
  - fromCurrency, toCurrency, rate, fetchedAt

PasskeyCredential:
  - credentialId, publicKey, signCount, createdAt
```

## Authentication

**Passkeys (WebAuthn)** - Modern passwordless authentication
- No user accounts - just credential registration
- Library: [Yubico java-webauthn-server](https://github.com/Yubico/java-webauthn-server)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Krever/ssbudget](https://github.com/Krever/ssbudget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->

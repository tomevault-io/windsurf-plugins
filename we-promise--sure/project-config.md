---
trigger: always_on
description: This rule explains the system architecture and data flow of the Rails app
---


This file outlines how the codebase is structured and how data flows through the app.

This is a personal finance application built in Ruby on Rails.  The primary domain entities for this app are outlined below.  For an authoritative overview of the relationships, [schema.rb](mdc:db/schema.rb) is the source of truth.

## App Modes

The codebase runs in two distinct "modes", dictated by `Rails.application.config.app_mode`, which can be `managed` or `self_hosted`.

- "Managed" - in managed mode, a team operates and manages servers for users
- "Self Hosted" - in self hosted mode, users host the codebase on their own infrastructure, typically through Docker Compose.  We have an example [docker-compose.example.yml](mdc:docker-compose.example.yml) file that runs [Dockerfile](mdc:Dockerfile) for this mode.

## Families and Users

- `Family` - all Stripe subscriptions, financial accounts, and the majority of preferences are stored at the [family.rb](mdc:app/models/family.rb) level.
- `User` - all [session.rb](mdc:app/models/session.rb) happen at the [user.rb](mdc:app/models/user.rb) level.  A user belongs to a `Family` and can either be an `admin` or a `member`.  Typically, a `Family` has a single admin, or "head of household" that manages finances while there will be several `member` users who can see the family's finances from varying perspectives.

## Currency Preference

Each `Family` selects a currency preference.  This becomes the "main" currency in which all records are "normalized" to via [exchange_rate.rb](mdc:app/models/exchange_rate.rb) records so that the app can calculate metrics, historical graphs, and other insights in a single family currency.

## Accounts

The center of the app's domain is the [account.rb](mdc:app/models/account.rb).  This represents a single financial account that has a `balance` and `currency`.  For example, an `Account` could be "Chase Checking", which is a single financial account at Chase Bank.  A user could have multiple accounts at a single institution (i.e. "Chase Checking", "Chase Credit Card", "Chase Savings") or an account could be a standalone account, such as "My Home" (a primary residence).

### Accountables

In the app, [account.rb](mdc:app/models/account.rb) is a Rails "delegated type" with the following subtypes (separate DB tables).  Each account has a `classification` or either `asset` or `liability`.  While the types are a flat hierarchy, below, they have been organized by their classification:

- Asset accountables
  - [depository.rb](mdc:app/models/depository.rb) - a typical "bank account" such as a savings or checking account
  - [investment.rb](mdc:app/models/investment.rb) - an account that has "holdings" such as a brokerage, 401k, etc.
  - [crypto.rb](mdc:app/models/crypto.rb) - an account that tracks the value of one or more crypto holdings
  - [property.rb](mdc:app/models/property.rb) - an account that tracks the value of a physical property such as a house or rental property
  - [vehicle.rb](mdc:app/models/vehicle.rb) - an account that tracks the value of a vehicle
  - [other_asset.rb](mdc:app/models/other_asset.rb) - an asset that cannot be classified by the other account types.  For example, "jewelry".
- Liability accountables
  - [credit_card.rb](mdc:app/models/credit_card.rb) - an account that tracks the debt owed on a credit card
  - [loan.rb](mdc:app/models/loan.rb) - an account that tracks the debt owed on a loan (i.e. mortgage, student loan)
  - [other_liability.rb](mdc:app/models/other_liability.rb) - a liability that cannot be classified by the other account types.  For example, "IOU to a friend"

### Account Balances

An account [balance.rb](mdc:app/models/account/balance.rb) represents a single balance value for an account on a specific `date`.  A series of balance records is generated daily for each account and is how we show a user's historical balance graph.  

- For simple accounts like a "Checking Account", the balance represents the amount of cash in the account for a date.  
- For a more complex account like "Investment Brokerage", the `balance` represents the combination of the "cash balance" + "holdings value".  Each accountable type has different components that make up the "balance", but in all cases, the "balance" represents "How much the account is worth" (when `classification` is `asset`) or "How much is owed on the account" (when `classification` is `liability`)

All balances are calculated daily by [balance_calculator.rb](mdc:app/models/account/balance_calculator.rb).

### Account Holdings

An account [holding.rb](mdc:app/models/holding.rb) applies to [investment.rb](mdc:app/models/investment.rb) type accounts and represents a `qty` of a certain [security.rb](mdc:app/models/security.rb) at a specific `price` on a specific `date`.

For investment accounts with holdings, [base_calculator.rb](mdc:app/models/holding/base_calculator.rb) is used to calculate the daily historical holding quantities and prices, which are then rolled up into a final "Balance" for the account in [base_calculator.rb](mdc:app/models/account/balance/base_calculator.rb).

### Account Entries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [we-promise/sure](https://github.com/we-promise/sure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->

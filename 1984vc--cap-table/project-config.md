---
trigger: always_on
description: > **Purpose:** Enable an agent to build an interactive cap-table calculator inside a chat interface.
---

# Cap Table Modeling Skill

> **Purpose:** Enable an agent to build an interactive cap-table calculator inside a chat interface.  
> **Source:** This skill documents the complete mathematical model, the annotated reference implementation, and conversation patterns used by the [`@1984vc/cap-table`](https://github.com/1984vc/cap-table) TypeScript library.

---

## Table of Contents

1. [Mathematical Foundations](#1-mathematical-foundations)
2. [Complete Annotated Implementation](#2-complete-annotated-implementation)
3. [Building the Conversational Tool](#3-building-the-conversational-tool)

---

## 1. Mathematical Foundations

### 1.1 Ownership Basics

At its core, a cap table is a list of ownership stakes. For any stakeholder:

```
ownershipPct = shares / totalShares
```

All percentages in the library are expressed as decimals (e.g., `0.45` = 45%).

### 1.2 Pre-Money vs Post-Money Valuation

When a priced round occurs, two valuations matter:

- **Pre-money valuation** = The value of the company *before* the new investment.
- **Post-money valuation** = Pre-money + total new investment.

The **price per share (PPS)** for the Series round is derived from the post-money:

```
PPS = postMoneyValuation / totalPostMoneyShares
```

Where `totalPostMoneyShares` includes:
- All existing common shares (founders + issued options)
- Any new shares issued to SAFE investors upon conversion
- Any increase in the options pool (the "refresh")
- Series investor shares

### 1.3 SAFE Conversion Mechanics

A SAFE (Simple Agreement for Future Equity) converts to shares at a priced round. The conversion depends on three variables:

1. **Cap** — the maximum valuation used for conversion
2. **Discount** — a percentage reduction off the Series PPS
3. **Conversion type** — whether the cap applies to pre-money or post-money share count

#### 1.3.1 Pre-Money SAFE

A pre-money SAFE converts based on the pre-money share count:

```
capPPS = cap / preMoneyShares
shares = investment / capPPS
```

Equivalently:

```
shares = (investment / cap) * preMoneyShares
```

The investor's ownership is dilutive — they get shares *before* the new money comes in, so the Series investors dilute them too.

#### 1.3.2 Post-Money SAFE

A post-money SAFE (the Y Combinator standard) guarantees a fixed ownership percentage of the *post-money* cap table:

```
ownershipPct = investment / cap
```

This means the SAFE investor's stake is calculated *after* all conversions but *before* the Series round. Critically, post-money SAFEs are **not diluted** by the Series investors — their ownership percentage is locked in at conversion.

#### 1.3.3 Discount

A discount gives the SAFE investor a lower PPS than the Series investors:

```
discountPPS = (1 - discount) * seriesPPS
```

For example, a 20% discount means `discountPPS = 0.80 * seriesPPS`.

#### 1.3.4 Effective Conversion Price

The SAFE investor always gets the **better** of the cap or the discount:

```
effectivePPS = min(discountPPS, capPPS)
shares = investment / effectivePPS
```

If the cap is `0` (uncapped), only the discount applies:

```
effectivePPS = (1 - discount) * seriesPPS
```

#### 1.3.5 MFN (Most Favored Nation)

An MFN SAFE has no explicit cap but receives the **lowest cap** of any subsequent capped SAFE. This is implemented by scanning all SAFEs that come *after* the MFN SAFE in the list and taking the minimum non-zero cap among post-money SAFEs.

```
mfnCap = min(cap of all subsequent post-money SAFEs where cap > 0)
```

If no subsequent capped SAFE exists, the MFN remains uncapped until one is added.

#### 1.3.6 YC 7% Post-Money

A special case: guarantees exactly 7% ownership post-conversion:

```
ownershipPct = 0.07
```

This is treated as a post-money SAFE with `cap = investment / 0.07`.


### 1.4 The Iterative Solver (`fitConversion`)

The central challenge in cap table math is that **SAFE conversions depend on the total share count, but the total share count depends on SAFE conversions**. This is a circular dependency that requires an iterative solver.

#### Why It's Circular

Consider a post-money SAFE:

```
ownershipPct = investment / cap   // fixed
shares = ownershipPct * totalShares // depends on totalShares
totalShares = commonShares + safeShares + seriesShares + optionsPool // depends on shares
```

You can't solve for `totalShares` algebraically in one step because of rounding and the interaction between pre-money and post-money SAFEs.

#### The Iteration

The solver starts with an initial guess:

```
totalShares = commonShares + unusedOptions
```

Then it repeatedly computes what the total shares *should* be given that guess, and uses the result as the next guess. The iteration converges when the guess stabilizes.

At each iteration, given a `totalShares` guess:

1. **Compute the refreshed options pool:**
   ```
   optionsPool = max(totalShares * targetOptionsPct, unusedOptions)
   ```
   The pool can't shrink below existing unused options.

2. **Compute the increase in the options pool:**
   ```
   increaseInOptionsPool = optionsPool - unusedOptions
   ```

3. **Compute the Series PPS:**
   ```
   seriesPPS = (preMoneyValuation + totalSeriesInvestment) / totalShares
   ```
   Note: the numerator is the post-money valuation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1984vc/cap-table](https://github.com/1984vc/cap-table) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

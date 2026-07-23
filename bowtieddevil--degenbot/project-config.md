---
trigger: always_on
description: This document captures lessons learned, architectural insights, and debugging guidance for the Degenbot CLI Aave processing system.
---

# Degenbot CLI - Agent Knowledge Base

This document captures lessons learned, architectural insights, and debugging guidance for the Degenbot CLI Aave processing system.

## Aave V3 Processing

### Interest Accrual - Critical Understanding

**Lesson:** Interest accrual Mint events are emitted for tracking purposes ONLY and do NOT increase the scaled balance.

**Context:** When processing Aave V3 transactions, the aToken contract emits Mint events during interest accrual (e.g., before transfers, withdrawals). However, these events are purely informational - they do not actually mint tokens or change the user's scaled balance.

**Contract Behavior (aToken rev_1.sol:2825-2855):**
```solidity
function _transfer(address sender, address recipient, uint256 amount, uint256 index) internal {
    // Calculate interest accrued (tracks interest earned since last interaction)
    uint256 senderBalanceIncrease = senderScaledBalance.rayMul(index) -
        senderScaledBalance.rayMul(_userState[sender].additionalData);
    
    // Update the stored index ONLY - this is the ONLY state change for interest accrual
    _userState[sender].additionalData = index.toUint128();
    
    // Transfer scaled balance
    super._transfer(sender, recipient, amount.rayDiv(index).toUint128());
    
    // Emit Mint event for TRACKING ONLY - no actual _mint() is called!
    // This event is emitted solely for off-chain tracking purposes.
    if (senderBalanceIncrease > 0) {
        emit Mint(_msgSender(), sender, senderBalanceIncrease, senderBalanceIncrease, index);
    }
}
```

**Key Insight:**
- Interest = `scaledBalance * (newIndex - oldIndex) / RAY`
- The user's **scaled balance** does not change
- The user's **effective balance** increases because the index increased
- Mint event `amount` field = interest in underlying units (for tracking)

**Implementation:**
```python
# In enrichment.py - INTEREST_ACCRUAL operations
if operation.operation_type.name == "INTEREST_ACCRUAL":
    raw_amount = scaled_event.amount  # Interest in underlying units
    scaled_amount = 0  # NO balance change - event is tracking-only
```

**Issue Reference:** `debug/aave/0004 - Interest Accrual Scaling Error in Enrichment.md`

---

### Pool Versions and Scaling

**Lesson:** Pool revisions and token revisions are independent versioning systems that affect different aspects of amount handling.

**Context:** Two separate versioned systems exist:

1. **Pool Revisions** (affects how amounts are passed between contracts):
   - **Pool revisions 1-8:** The Pool contract passes unscaled amounts (underlying units) to the token contract
   - **Pool revision 9+:** The Pool contract pre-calculates scaled amounts before calling the token contract

2. **Token Revisions** (affects rounding math behavior and processor selection):
   - **Token revisions 1-3:** Uses `HalfUpRoundingMath` - standard `ray_div` with half-up rounding
   - **Token revision 4+:** Uses `ExplicitRoundingMath` - explicit floor/ceil rounding

**Python Implementation:**
The enrichment layer handles both systems independently:
1. Pool revision determines if raw amounts need scaling before processing (for pool rev 9+)
2. Token revision determines which processor and rounding math to use via factories

**Processor mapping (by token revision):**
```python
# TokenProcessorFactory.COLLATERAL_PROCESSORS
{
    1: CollateralV1Processor,  # HalfUpRoundingMath
    2: CollateralV1Processor,  # Same as rev 1
    3: CollateralV3Processor,  # HalfUpRoundingMath
    4: CollateralV4Processor,  # ExplicitRoundingMath
    5: CollateralV5Processor,  # ExplicitRoundingMath
}

# TokenProcessorFactory.DEBT_PROCESSORS
{
    1: DebtV1Processor,
    2: DebtV1Processor,  # Same as rev 1
    3: DebtV3Processor,
    4: DebtV4Processor,
    5: DebtV5Processor,
}

# TokenProcessorFactory.GHO_DEBT_PROCESSORS
{
    1: GhoV1Processor,  # No discount
    2: GhoV2Processor,  # Discount support
    3: GhoV2Processor,   # Same as rev 2
    4: GhoV4Processor,  # Discount deprecated
    5: GhoV5Processor,  # Explicit rounding
    6: GhoV5Processor,   # Same as rev 5
}
```

**TokenMath mapping (by pool version):**
```python
# TokenMathFactory._TOKEN_MATH
{
    1: HalfUpRoundingMath,   # Pool revs 1-3
    2: HalfUpRoundingMath,
    3: HalfUpRoundingMath,
    4: ExplicitRoundingMath, # Pool revs 4-10
    5: ExplicitRoundingMath,
    6: ExplicitRoundingMath,
    7: ExplicitRoundingMath,
    8: ExplicitRoundingMath,
    9: ExplicitRoundingMath,
    10: ExplicitRoundingMath,
}
# Use get_token_math_for_token_revision(revision) to map token revision → pool version
```

**Key Point:** Token revision determines both the processor and rounding math. Pool and token revisions typically move together but are technically independent. Use `TokenProcessorFactory` to get the correct processor and `TokenMathFactory` for math operations.

---

### Math Library Architecture

**Lesson:** Four levels of abstraction separate concerns: primitives, math libraries, processors, and enrichment.

**Architecture:**

The Aave module uses a layered architecture:

1. **`wad_ray_math.py`** - Low-level primitives
   - `ray_mul`, `ray_div` - Half-up rounding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BowTiedDevil/degenbot](https://github.com/BowTiedDevil/degenbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

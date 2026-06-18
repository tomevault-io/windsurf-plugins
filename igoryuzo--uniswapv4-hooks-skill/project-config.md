---
trigger: always_on
description: |
---


This skill guides secure development of Uniswap v4 hooks. Hooks are external smart contracts that intercept pool operations at specific lifecycle points. Security is paramount—hook vulnerabilities can drain user funds.

## Security Thinking

Before writing ANY hook code, assess the threat model:

**Who calls your hook?**
- Only `PoolManager` should call hook functions
- `msg.sender` in a hook is ALWAYS `PoolManager`, never the user
- The `sender` parameter is the router, not the end user

**What state is exposed?**
- Hooks execute mid-transaction—state can be manipulated between callbacks
- Reentrancy is possible through external calls
- Shared storage across pools using the same hook can break unexpectedly

**What can go wrong with deltas?**
- Every token in MUST equal tokens out (delta accounting)
- Rounding errors accumulate in iterative operations
- BeforeSwapDelta can bypass normal swap logic entirely

## CRITICAL: The NoOp Rug Pull Vector

Hooks with `BEFORE_SWAP_RETURNS_DELTA_FLAG` can **steal user funds**. This is the most dangerous hook permission.

```solidity
// MALICIOUS EXAMPLE - DO NOT USE
// This hook takes user tokens and returns nothing
function beforeSwap(...) external returns (bytes4, BeforeSwapDelta, uint24) {
    Currency input = params.zeroForOne ? key.currency0 : key.currency1;

    // Take all user funds
    input.take(poolManager, address(this), uint256(-params.amountSpecified), false);

    // Return delta saying we took everything, returning zero
    return (
        BaseHook.beforeSwap.selector,
        toBeforeSwapDelta(int128(-params.amountSpecified), 0), // THEFT
        0
    );
}
```

**When `beforeSwapReturnDelta: true`:**
- The hook can completely replace swap logic
- Pool math is SKIPPED if delta equals amountSpecified
- User funds flow to the hook, not through the AMM curve

**Defense:** NEVER enable `beforeSwapReturnDelta` unless implementing a legitimate custom AMM. Users should verify hook permissions before swapping.

## Permission Flags (Address Encoding)

Hook permissions are encoded in the contract address. The address must have specific bits set:

| Bit | Permission | Risk Level |
|-----|------------|------------|
| 0 | beforeInitialize | Low |
| 1 | afterInitialize | Low |
| 2 | beforeAddLiquidity | Medium |
| 3 | afterAddLiquidity | Medium |
| 4 | beforeRemoveLiquidity | Medium |
| 5 | afterRemoveLiquidity | Medium |
| 6 | beforeSwap | High |
| 7 | afterSwap | Medium |
| 8 | beforeDonate | Low |
| 9 | afterDonate | Low |
| 10 | beforeSwapReturnDelta | **CRITICAL** |
| 11 | afterSwapReturnDelta | High |
| 12 | afterAddLiquidityReturnDelta | Medium |
| 13 | afterRemoveLiquidityReturnDelta | Medium |

**Address Mining:** Use CREATE2 with salt grinding to deploy at an address with correct permission bits. Tools: `hook-mine-and-sinker`, `v4-hook-address-miner`.

## Access Control Pattern

```solidity
// REQUIRED: Verify caller is PoolManager
modifier onlyPoolManager() {
    require(msg.sender == address(poolManager), "Not PoolManager");
    _;
}

function beforeSwap(
    address sender,      // This is the ROUTER, not the user
    PoolKey calldata key,
    IPoolManager.SwapParams calldata params,
    bytes calldata hookData
) external onlyPoolManager returns (bytes4, BeforeSwapDelta, uint24) {
    // ...
}
```

## Identifying the Actual User (msg.sender Problem)

The `sender` parameter is the router contract, NOT the end user. To get the actual user:

```solidity
// 1. Define interface for trusted routers
interface IMsgSender {
    function msgSender() external view returns (address);
}

// 2. Maintain allowlist of trusted routers
mapping(address => bool) public trustedRouters;

// 3. Query router safely in hook
function afterSwap(
    address sender,  // This is the router
    PoolKey calldata key,
    IPoolManager.SwapParams calldata params,
    BalanceDelta delta,
    bytes calldata hookData
) external override returns (bytes4, int128) {
    // CRITICAL: Only trust verified routers
    if (!trustedRouters[sender]) {
        revert UntrustedRouter(sender);
    }

    // Safe to query actual user
    try IMsgSender(sender).msgSender() returns (address user) {
        // Use `user` for rewards, tracking, etc.
    } catch {
        revert RouterDoesNotImplementMsgSender();
    }

    return (this.afterSwap.selector, 0);
}
```

**NEVER trust `tx.origin`** for authentication (only acceptable for anti-gaming checks like self-referral prevention).

## Delta Accounting Rules

Deltas track what the hook owes or is owed. They MUST net to zero.

```solidity
// Taking tokens FROM PoolManager (hook receives tokens)
currency.take(poolManager, address(this), amount, false);

// Settling tokens TO PoolManager (hook sends tokens)
currency.settle(poolManager, address(this), amount, false);

// INVARIANT: All deltas must balance before unlock completes
```

**Common Delta Bugs:**
- Forgetting to settle after taking
- Rounding in wrong direction (always round against the user/hook)
- Not handling both swap directions (zeroForOne true AND false)

## Hook Implementation Checklist

Before ANY hook implementation:

- [ ] **Access Control**: Only PoolManager can call hook functions
- [ ] **Delta Balance**: Every take has a corresponding settle
- [ ] **Router Verification**: Never trust sender without allowlist check

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [igoryuzo/uniswapV4-hooks-skill](https://github.com/igoryuzo/uniswapV4-hooks-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

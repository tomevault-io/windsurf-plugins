---
trigger: always_on
description: - **Think first, code second**: Minimize the number of lines changed and consider ripple effects across the codebase.
---

## Solidity Contribution Guidelines

### 1. General Principles

- **Think first, code second**: Minimize the number of lines changed and consider ripple effects across the codebase.
- **Prefer simplicity**: Fewer moving parts ➜ fewer bugs and lower audit overhead.

### 2. Assembly Usage

| Rule | Rationale |
|------|-----------|
| Use assembly only when essential. | Keeps code readable and auditable. |
| Assembly is mandatory for low-level external calls. | Gives full control over call parameters & return data, and saves gas. |
| Precede every assembly block with: • A brief justification (1-2 lines). • Equivalent Solidity pseudocode. | Documents intent for reviewers. |
| Mark assembly blocks memory-safe when the Solidity docs' criteria are met. | Enables compiler optimizations. |

### 3. Gas Optimization

- Keep a dedicated **Gas Optimization** section in the PR description; justify any measurable gas deltas.
- Prefer `calldata` over `memory` for function arguments wherever possible, as `calldata` is cheaper. Note that `calldata` is read-only.
- Limit storage (`sstore`, `sload`) operations; cache in memory wherever possible.
- Use forge snapshot, forge test --match-test "benchmark", and npm scripts:
  ```bash
  npm run snapshot:main   # captures gas baseline from main
  npm run diff:main       # compares your branch vs. main
  ```
- Large regressions must be explained.

### 4. Handling "Stack Too Deep"

- **Struct hack (tests only)**: Bundle local variables into a temporary struct declared above the test.
- **Scoped blocks**: Wrap code in `{ ... }` to drop unused vars from the stack.
- **Internal helper functions**: Encapsulate logic to shorten call frames.
- **Refactor / delete unnecessary variables before other tricks**.

### 5. Security Checklist

- Review every change with an adversarial mindset.
- Favor the simplest design that meets requirements.
- After coding, ask: "What new attack surface did I introduce?"
- Reject any change that raises security risk without strong justification.

### 6. Reentrancy Protection

**All external functions MUST be protected against reentrancy attacks**. This is critical for maintaining contract security.

#### Checks-Effects-Interactions Pattern

Always follow the Checks-Effects-Interactions (CEI) pattern:

```solidity
// ✅ Correct: CEI pattern
function withdraw(uint256 amount) external {
    // 1. Checks
    if (balances[msg.sender] < amount) revert InsufficientBalance();
    
    // 2. Effects (state changes BEFORE external calls)
    balances[msg.sender] -= amount;
    
    // 3. Interactions (external calls LAST)
    (bool success,) = msg.sender.call{value: amount}("");
    if (!success) revert TransferFailed();
}

// ❌ Wrong: State change after external call
function withdrawUnsafe(uint256 amount) external {
    if (balances[msg.sender] < amount) revert InsufficientBalance();
    
    (bool success,) = msg.sender.call{value: amount}("");
    if (!success) revert TransferFailed();
    
    balances[msg.sender] -= amount; // VULNERABLE: State change after external call
}
```

#### When CEI Pattern Isn't Sufficient

If the Checks-Effects-Interactions pattern cannot be applied (e.g., complex multi-step operations), use a reentrancy guard:

```solidity
// Add reentrancy guard modifier when CEI pattern isn't possible
modifier nonReentrant() {
    if (_locked != 1) revert ReentrancyGuardReentrantCall();
    _locked = 2;
    _;
    _locked = 1;
}

function complexOperation() external nonReentrant {
    // Complex logic that requires multiple external calls
    // Protected by reentrancy guard
}
```

#### Key Rules for Reentrancy Safety

1. **Default to CEI pattern**: This should be your first choice for all functions
2. **State changes before calls**: Update all state variables before making external calls
3. **Use reentrancy guards sparingly**: Only when CEI pattern is genuinely not applicable
4. **Review all external calls**: Any `.call()`, `.transfer()`, `.send()`, or calls to other contracts
5. **Consider read-only reentrancy**: Even view functions called during state changes can be attack vectors
6. **Test reentrancy scenarios**: Write tests that attempt reentrancy attacks on your functions

### 7. Verification Workflow

```bash
forge build                    # compile
forge test                     # full test suite
forge snapshot                 # gas snapshot (local)
forge test --match-test bench  # run benchmarks
npm run snapshot:main          # baseline gas (main)
npm run diff:main              # gas diff vs. main
```

### 8. Continuous Learning

- Consult official Solidity docs and relevant project references when uncertain.
- Borrow battle-tested patterns from audited codebases.

Apply these rules rigorously before opening a PR.


### Error Handling Style

Always use custom errors with the revert pattern instead of require statements:

```solidity
// ❌ Don't use require with string messages
require(amount > 0, "Amount must be positive");
require(to != address(0), "Cannot transfer to zero address");

// ✅ Do use custom errors with if/revert pattern
error AmountMustBePositive();
error CannotTransferToZeroAddress();

if (amount == 0) revert AmountMustBePositive();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ithacaxyz/account](https://github.com/ithacaxyz/account) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->

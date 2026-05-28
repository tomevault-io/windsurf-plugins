---
trigger: always_on
description: description: Guidelines for writing clean, maintainable, and human-readable Solidity code. Apply these rules when writing or reviewing Solidity code to ensure consistency and quality.
---

---
description: Guidelines for writing clean, maintainable, and human-readable Solidity code. Apply these rules when writing or reviewing Solidity code to ensure consistency and quality.
globs: **/*.sol
---
# Solidity Smart Contracts Coding Style and Standards Guide

## Minimize Indentation/Nesting
- Try to minimize indentation within a function.
- Move nested conditionals into well-named functions.
- See examples in @.cursor/rule-examples/minimize-indentation.md

## Exit Early
- Exit as early as possible (i.e. `return`, `revert`, `break`, `continue`).
- See examples in @.cursor/rule-examples/exit-early.md

## Events Describe What Happened (Not What Changed)
- Events should be used to describe things that occurred during the contract's lifetime, not what values changed. For example, `Transfer` events rather than `BalanceUpdated` or `BalanceIncreased`/`BalanceDecreased` events. Similarly, `Deposit`, `Withdrawal`, `FeeCharged` events, rather than `BalanceUpdated` or `BalanceIncreased`/`BalanceDecreased` events.
- Old or previous values of arguments should never be part of the event. For exmaple, if there is an `AdminSet` event, it should emit the value of the new admin, an should not also emit the value of the previous admin.

## Minimize Source of Event Emission
- Try to minimize where a unique event is emitted from (ideally to only 1 place).
- See examples in @.cursor/rule-examples/minimize-event-source.md

## Checks, Effects, Interactions
- Whenever possible, perform all checks first (i.e. validation, requires/reverts, etc), then perform all effects (i.e. state changes, event emissions, etc), and then perform all interactions with other contracts.
- Take care to consider that calling internal state-changing functions may (either now, in the future, or via overrides) interact with other contracts.

## Avoid Booleans As Function Arguments
- Avoid defining functions with booleans as arguments that branch the logic within the function.
- Functions should not have two behaviours defined by a bool argument. If it is the case (i.e. `createUser(bool isPowerUser)`), there should be two functions instead (i.e. `createUser()` and `createPowerUser()`).
- It is acceptable to pass booleans as arguments only for the purpose of them being treated as opaque data that for an object property that will be created or stored in storage or memory.

## Avoid Private Visibility
- Avoid defining `private` variables and functions, and instead opt for `internal`.

## Avoid Declaring Interface Variables
- Avoid declaring state variables, function arguments, function returns, and local variables as interfaces.
- Instead, when inetracting with a contract at an address is neccessary, cast the address as an interface inline (i.e. `IERC20(token).balanceOf(aacount)`).

## Avoid Declaring Structs as Memory
- Structs should be declared as `storage` unless copying it entirely to memory is required.

## Explicit Import From Where Defined
- When importing, specify the exact set of resources being imported from the file.
- Avoid importing from files where the resource is not defined, but rather re-exported.

## Avoid Explicit Emitting Current Block Properties in Events
- Avoid explicitly emitting the blockhash, block number, block timestamp, chain id, tx origin, etc in events.

## Use Subset Interfaces for Interactions
- Define and use subset interfaces when interacting with other smart contracts, even if they are first party contracts.
- See examples in @.cursor/rule-examples/subset-interfaces.md

## Avoid Inline Magic/Literal Values
- Replace hard-coded values with named constants
- Use descriptive yet short constant names that explain the value's purpose
- Keep constants at the top of the file
- For obvious values (such as zero checks or type max checks) this rule can be ignored.

## Real Unit Tests
- A unit test should only call the contract for the exact function being tested, and no other functions, other than getters that simply return a state variable (in order to make assertions about state changes).
- Harnesses should be used to set up and access state as needed by the unit test.
- The unit tests should encompass all explicit expectations, which includes expected interactions, expected event emissions, and expected state changes.
- If there are internal variables that are expected, they should be accessed via a harness.
- See examples in @.cursor/rule-examples/unit-tests.md

## Public/Notice Natspec in Interface
- All `@notice` natspec should live in the interface of an implementation.
- `@dev` natspec intended for developers can exist anywhere, and should be used for, at the very least, a minimal explanation of internal functions and variables.

## Comprehensive Interface
- All contracts should implement an interface of the same name, which is a comprehensive set of all functions, including those inherited.
- See examples in @.cursor/rule-examples/comprehensive-interface.md

## Simple and Generic Variable Names

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xmtp/smart-contracts](https://github.com/xmtp/smart-contracts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->

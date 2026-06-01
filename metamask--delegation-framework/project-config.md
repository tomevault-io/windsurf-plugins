---
trigger: always_on
description: This guide focuses on creating smart contracts that work seamlessly with the MetaMask Delegation Toolkit. The key principle is to keep your contracts simple, focused on core functionality, and completely unaware of the delegation system itself.
---

# Developing Smart Contracts for Delegation Systems

This guide focuses on creating smart contracts that work seamlessly with the MetaMask Delegation Toolkit. The key principle is to keep your contracts simple, focused on core functionality, and completely unaware of the delegation system itself.

## Core Principles

1. **Simplicity**: Contracts should focus solely on their core business logic.
2. **Owner-centric**: Use `onlyOwner` modifiers for privileged functions.
3. **Delegation-agnostic**: Contracts should not reference Delegation, DelegationManager, or mode encoding.
4. **Extensibility**: Design core functions to be easily extended through the delegation framework.

## Contract Structure

Here's an example of a basic contract structure:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyContract is ERC721, Ownable {
    constructor(string memory name, string memory symbol) ERC721(name, symbol) Ownable(msg.sender) {}

    function mint(address to, uint256 tokenId) public onlyOwner {
        _mint(to, tokenId);
    }
}
```

## Core Functions

### Minting

The `mint` function is a simple example of a core function that can be easily extended through the delegation framework.

## Using Caveat Enforcers

Caveat enforcers allow you to add specific conditions or restrictions to delegations. The MetaMask Delegation Toolkit provides several out-of-the-box caveat enforcers:

- `AllowedCalldataEnforcer.sol`
- `AllowedMethodsEnforcer.sol`
- `AllowedTargetsEnforcer.sol`
- `BlockNumberEnforcer.sol`
- `DeployedEnforcer.sol`
- `ERC20TransferAmountEnforcer.sol`
- `ERC20BalanceChangeEnforcer.sol`
- `NonceEnforcer.sol`
- `LimitedCallsEnforcer.sol`
- `IdEnforcer.sol`
- `TimestampEnforcer.sol`
- `ValueLteEnforcer.sol`

So any policy that is composed of those can be assumed provided already.

In the case that you need to create a custom enforcer, you can use the `CaveatEnforcer.sol` base class and write your own like this:

```solidity
// SPDX-License-Identifier: MIT AND Apache-2.0
pragma solidity 0.8.23;

import "forge-std/Test.sol";
import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";
import { ModeLib } from "@erc7579/lib/ModeLib.sol";
import { ExecutionLib } from "@erc7579/lib/ExecutionLib.sol";

import { Execution, Caveat, Delegation, ModeCode } from "../../src/utils/Types.sol";
import { Counter } from "../utils/Counter.t.sol";
import { CaveatEnforcerBaseTest } from "./CaveatEnforcerBaseTest.t.sol";
import { AllowedMethodsEnforcer } from "../../src/enforcers/AllowedMethodsEnforcer.sol";
import { ICaveatEnforcer } from "../../src/interfaces/ICaveatEnforcer.sol";

contract AllowedMethodsEnforcerTest is CaveatEnforcerBaseTest {

    ////////////////////// State //////////////////////

    AllowedMethodsEnforcer public allowedMethodsEnforcer;
    ModeCode public mode = ModeLib.encodeSimpleSingle();

    ////////////////////// Set up //////////////////////

    function setUp() public override {
        super.setUp();
        allowedMethodsEnforcer = new AllowedMethodsEnforcer();
        vm.label(address(allowedMethodsEnforcer), "Allowed Methods Enforcer");
    }

    ////////////////////// Valid cases //////////////////////

    // should allow a method to be called when a single method is allowed
    function test_singleMethodCanBeCalled() public {
        // Create the execution that would be executed
        Execution memory execution_ = Execution({
            target: address(aliceDeleGatorCounter),
            value: 0,
            callData: abi.encodeWithSelector(Counter.increment.selector)
        });
        bytes memory executionCallData_ = ExecutionLib.encodeSingle(execution_.target, execution_.value, execution_.callData);

        // beforeHook, mimicking the behavior of Alice's DeleGator
        vm.prank(address(delegationManager));
        allowedMethodsEnforcer.beforeHook(
            abi.encodePacked(Counter.increment.selector), hex"", mode, executionCallData_, keccak256(""), address(0), address(0)
        );
    }

    // should allow a method to be called when a multiple methods are allowed
    function test_multiMethodCanBeCalled() public {
        // Create the execution that would be executed
        Execution memory execution_ = Execution({
            target: address(aliceDeleGatorCounter),
            value: 0,
            callData: abi.encodeWithSelector(Counter.increment.selector)
        });
        bytes memory executionCallData_ = ExecutionLib.encodeSingle(execution_.target, execution_.value, execution_.callData);

        // beforeHook, mimicking the behavior of Alice's DeleGator
        vm.prank(address(delegationManager));
        allowedMethodsEnforcer.beforeHook(
            abi.encodePacked(Counter.setCount.selector, Ownable.renounceOwnership.selector, Counter.increment.selector),
            hex"",
            mode,
            executionCallData_,
            keccak256(""),
            address(0),
            address(0)
        );
    }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MetaMask/delegation-framework](https://github.com/MetaMask/delegation-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->

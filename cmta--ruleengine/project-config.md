---
trigger: always_on
description: This file helps AI agents (Cursor, Claude Code, etc.) understand and work with this codebase.
---

# AI Agent Guide for RuleEngine

This file helps AI agents (Cursor, Claude Code, etc.) understand and work with this codebase.

AGENTS.md and CLAUDE.md files must always be identical

## Project Summary

**RuleEngine** is a Solidity smart contract system that enforces transfer restrictions for [CMTAT](https://github.com/CMTA/CMTAT) and [ERC-3643](https://eips.ethereum.org/EIPS/eip-3643) tokens. It acts as an external controller that calls pluggable rule contracts on each token transfer, mint, or burn.

- **Version:** 3.0.0 (defined in `src/modules/VersionModule.sol`)
- **Solidity:** ^0.8.20 (compiled with 0.8.34)
- **EVM target:** Prague
- **License:** MPL-2.0

## Build & Test Commands

```bash
forge build              # Compile all contracts
forge test               # Run all tests
forge test -vvv          # Verbose test output
forge test --match-contract <Name> --match-test <fn>  # Run specific test
forge coverage           # Code coverage
forge coverage --no-match-coverage "(script|mocks|test)" --report lcov  # Production coverage
forge fmt                # Format code
```

Dependencies are git submodules. Initialize with `forge install`, update with `forge update`.
CMTAT submodule also needs `cd lib/CMTAT && npm install` for its OpenZeppelin deps.

## Import Remappings

| Alias | Path |
|-------|------|
| `CMTAT/` | `lib/CMTAT/contracts/` |
| `CMTATv3.0.0/` | `lib/CMTATv3.0.0/contracts/` |
| `@openzeppelin/contracts/` | `lib/openzeppelin-contracts/contracts` |

Use `@openzeppelin/contracts/` for OpenZeppelin imports, `CMTAT/` for CMTAT imports, `src/` for local imports.

## Architecture

### Three Deployable Contracts

```
RuleEngine              — RBAC via AccessControl (multi-operator)
RuleEngineOwnable       — ERC-173 Ownable (single-owner)
RuleEngineOwnable2Step  — ERC-173 Ownable2Step (single-owner, two-step handover)
```

All three share their core logic through `RuleEngineBase` directly or via `RuleEngineOwnableShared`.

### Inheritance Hierarchy

```
RuleEngineBase (abstract)
├── VersionModule                         → version() returns "3.0.0"
├── RulesManagementModule                 → add/remove/set/clear rules
│   ├── AccessControl (OZ)
│   └── RulesManagementModuleInvariantStorage  → errors, events, roles
├── ERC3643ComplianceModule               → bind/unbind tokens
│   └── IERC3643Compliance
├── RuleEngineInvariantStorage            → errors
└── IRuleEngineERC1404                    → CMTAT interface

RuleEngine
├── ERC2771ModuleStandalone → gasless support
└── RuleEngineBase

RuleEngineOwnable
├── ERC2771ModuleStandalone → gasless support
├── RuleEngineOwnableShared
│   └── RuleEngineBase
└── Ownable (OZ) → ERC-173

RuleEngineOwnable2Step
├── ERC2771ModuleStandalone → gasless support
├── RuleEngineOwnableShared
│   └── RuleEngineBase
└── Ownable2Step (OZ) → ERC-173
```

### Access Control Pattern

Modules define **virtual internal hooks** for access control. Concrete contracts override them:

```solidity
// In RulesManagementModule (abstract):
function _onlyRulesManager() internal virtual;

// In ERC3643ComplianceModule (abstract):
function _onlyComplianceManager() internal virtual;

// RuleEngine overrides with RBAC:
function _onlyRulesManager() internal virtual override onlyRole(RULES_MANAGEMENT_ROLE) {}
function _onlyComplianceManager() internal virtual override onlyRole(COMPLIANCE_MANAGER_ROLE) {}

// RuleEngineOwnable overrides with Ownable:
function _onlyRulesManager() internal virtual override onlyOwner {}
function _onlyComplianceManager() internal virtual override onlyOwner {}
```

**When adding a new protected function**, follow this pattern: define a virtual hook in the module, then override it in `RuleEngine`, `RuleEngineOwnable`, and `RuleEngineOwnable2Step`.

### `_checkRule` Override Chain

Rule validation uses a two-layer override:

1. **`RulesManagementModule._checkRule()`** — checks zero address + duplicates
2. **`RuleEngineBase._checkRule()`** — calls `super._checkRule()` then validates ERC-165 interface

```solidity
// RulesManagementModule (generic checks):
function _checkRule(address rule_) internal view virtual {
    if (rule_ == address(0x0)) revert ...ZeroNotAllowed();
    if (_rules.contains(rule_)) revert ...AlreadyExists();
}

// RuleEngineBase (adds ERC-165 check):
function _checkRule(address rule_) internal view virtual override {
    super._checkRule(rule_);
    if (!ERC165Checker.supportsInterface(rule_, RuleInterfaceId.IRULE_INTERFACE_ID))
        revert RuleEngine_RuleInvalidInterface();
}
```

### Rule Execution Flow

```
Token.transfer() → RuleEngine.transferred(from, to, value)
                    ├── onlyBoundToken modifier (caller must be bound)
                    └── for each rule in _rules:
                          rule.transferred(from, to, value)  // reverts if disallowed
```

View path: `detectTransferRestriction()` iterates rules, returns first non-zero code.

### Storage: EnumerableSet

Both rules and bound tokens use `EnumerableSet.AddressSet`:
- `_rules` in `RulesManagementModule` — the set of active rules
- `_boundTokens` in `ERC3643ComplianceModule` — tokens allowed to call `transferred`

This gives O(1) add/remove/contains and iterable storage.

## Key Interfaces


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CMTA) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->

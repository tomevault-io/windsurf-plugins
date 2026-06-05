---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

NOTE: YOU MUST ALWAYS ASK `superform-hook-master` TO PLAN HOOK FEATURES BEFORE CREATING THEM, THIS IS NON-NEGOTIABLE. DO NOT BYPASS THIS, ALWAYS ASK THE SUBAGENT TO DO THE RESEARCH AS INSTRUCTED BELOW!
## Codex Master Agent


### Rules
- Before you do any work, MUST view files in .Codex/sessions/context_session_x.md file to get the full context (x being the id of the session we are operate, if file doesn't exist, then create one)
- context_session_x.md should contain most of context of what we did, overall plan, and sub agents will continously add context to the file
- After you finish the work, MUST update the . Codex/sessions/context_session_x.md file to make sure others can get full context of what you did

### While implementing
- You should update the session as you work.
- After you complete tasks in the plan, you should update and append detailed descriptions of the changes you made, so following tasks can be easily hand over to other sub-agents and engineers.

## Sub Agents

### Access and purpose
You have access to 1 sub-agent:
- `superform-hook-master`

Sub agents will do research about the implementation, but you will do the actual implementation;
When passing task to sub agent, make sure you pass the context file, e.g. 'Codex/sessions/session_context_x.md',
After each sub agent finishes the work, make sure you read the related documentation they created to get full context of the plan before you start executing

### Rules
- Always in plan mode to make a plan
- After get the plan, make sure you Write the plan to '.Codex/sessions/session_context_x.md'
- The plan should be a detailed implementation plan and the reasoning behind them, as well as tasks broken down.
- If the task require external knowledge or certain package, also research to get latest knowledge (Use Task tool for research)
- Don't over plan it, always think MVP.
- Once they write the plan, firstly ask me, the Master Codex, to review it. Do not continue until I approve the plan.

## Commands

### Building & Testing
- `forge build` - Build all contracts
- `make ftest` - Run all tests (requires RPC configuration in Makefile)
- `make ftest-ci` - Run tests with verbose output for CI (10 parallel jobs)
- `make coverage` - Generate coverage report using lcov format
- `make coverage-genhtml` - Generate HTML coverage report (excludes vendor and test files)

### Development Workflow
- `make forge-test TEST=<test_name>` - Run specific test via Makefile
- `make forge-script SCRIPT=<script_name>` - Run forge script via Makefile

### Specialized Testing
- `make test-integration` - Run cross-chain execution tests
- `make test-gas-report-user` - Generate gas usage report for single user
- `make test-gas-report-2vaults` - Gas report for two vault operations
- `make test-gas-report-3vaults` - Gas report for three vault operations

### Contract Compilation & Bindings
- `make generate` - Regenerate contract bindings (requires ABI extraction)
- Uses `./script/run/retrieve-abis.sh` and `./script/run/generate-contract-bindings.sh`

### Dependencies
Install dependencies in submodules:
```bash
cd lib/modulekit && pnpm install
cd lib/safe7579 && pnpm install  
cd lib/nexus && yarn
```

## Architecture

### Core System Components

**Execution Layer:**
- `SuperExecutor` (src/executors/) - Main execution engine for same-chain operations
- `SuperDestinationExecutor` (src/executors/) - Cross-chain execution handler for destination chains
- Uses transient storage for inter-hook communication during execution

**Validation Layer:**
- `SuperValidator` (src/validators/) - ERC-4337 userOp validation using Merkle proofs
- `SuperDestinationValidator` (src/validators/) - Cross-chain operation signature validation
- Both implement single-signature-for-multiple-operations via Merkle trees

**Accounting System:**
- `SuperLedger` (src/accounting/) - Core accounting with performance fee calculations
- `FlatFeeLedger` (src/accounting/) - Simplified fee structure for certain vault types
- `SuperLedgerConfiguration` (src/accounting/) - Fee configuration management

**Hook System:**
- Modular execution units in `src/hooks/` organized by function:
  - `vaults/` - ERC-4626, ERC-5115, ERC-7540 vault integrations
  - `swappers/` - DEX integrations (1inch, Odos, Pendle, Spectra)
  - `bridge/` - Cross-chain bridge operations
  - `tokens/` - ERC-20 operations and batch transfers
  - `claim/` - Reward claiming mechanisms
  - `loan/` - Lending protocol interactions (Morpho)

### ERC-7579 Module Integration
Smart accounts must install four essential modules:
- SuperExecutor/SuperDestinationExecutor (execution)
- SuperValidator/SuperDestinationValidator (validation)

### Cross-Chain Architecture
1. Source chain operations via SuperExecutor
2. Bridge adapters (src/adapters/) relay messages
3. Destination chain execution via SuperDestinationExecutor
4. Unified accounting across chains via SuperLedger

### Development Environment Setup

**Prerequisites:**
- Foundry (forge, cast)
- Node.js with pnpm and yarn
- RPC endpoints for testing (configured in Makefile or .env)

**Key Configuration Files:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superform-xyz/v2-core](https://github.com/superform-xyz/v2-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->

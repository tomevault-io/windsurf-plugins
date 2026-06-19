---
trigger: always_on
description: Use this skill when the user wants to write a Foundry Proof of Concept (PoC) test that reproduces a smart contract vulnerability against a real deployed protocol on a mainnet fork. Triggers include phrases like "write a PoC", "reproduce this bug with Foundry", "fork mainnet and exploit", "validate this finding on-chain", or when the user provides a vulnerability report alongside deployed contract addresses. This skill is strictly for mainnet-forked, real-contract, end-to-end reproductions on EVM
---


# Foundry PoC on Mainnet Fork

## Purpose

Produce one Foundry test file that reproduces a real smart contract vulnerability against real deployed contracts on a forked EVM network. The test must pass, and its passing must prove the vulnerability end-to-end: from the action that first triggers the vulnerable state, through every on-chain step in between, to the final realized impact.

## Reading Order (Mandatory)

Before reading anything else in the repository, read the finding description to completion. Only after the finding is fully read should other files be consulted. In particular:

- Do not read any file in the target PoC directory (typically `test/poc/`) until the finding is classified and the causal chain is written out (see Procedure).
- If a file with the planned output name already exists in the repo, treat it as untrusted until the skill's own classification is complete. Stale or prior-session PoC files may encode causal-chain choices that violate the skill's rules. Do not anchor on them.
- Reference tests for style matching (imports, inheritance, helpers) are read in step 6 of the procedure, not before. Even then, imitate their surface style only. Never imitate their fork-block choice, starting actor, or causal-chain structure without independently verifying those match the current finding's classification.

The order in which files are read shapes Claude's reasoning. Reading an existing PoC before classifying the finding anchors output to that PoC's structure and overrides the skill's rules in practice. Reading the finding first is not optional.

## Required Inputs

Before writing any code, confirm the user has provided:

1. Vulnerability description: root cause, attack path, impact. If the description is short or summary-level, ask for the full finding.
2. Chain: ethereum, arbitrum, base, optimism, polygon, bnb, avalanche, or chain ID.
3. Fork target: `latest`, a specific block number, or a timestamp (if timestamp, ask the user to convert it to a block number, do not guess).
4. Real deployed addresses for every contract in the attack path, labeled by role.

Optional but improves output quality:

5. RPC URL (otherwise use a public default and state which one was used). For pinned block numbers, the RPC must have historical state for that block. Public RPCs like publicnode often lack archive data; drpc.org, mevblocker.io, and eth-pokt.nodies.app retain broader history. If a chosen RPC returns "historical state not available", try a different public endpoint before asking the user for an archive-enabled RPC.
6. Path to the repo's existing test files for style matching.
7. Severity (proceed for High, Medium, Critical; for Low or Info, warn the user that a PoC may not be warranted and confirm before proceeding).

If any of 1, 2, 3, or 4 is missing, stop and ask. Do not guess addresses. Do not assume "mainnet" means Ethereum if the protocol could be on another chain. Do not fabricate a fork block.

## Hard Rules

### Rule 1: Foundry Only

Foundry is the only acceptable framework. If the target repo uses Hardhat, Truffle, or anything else, the PoC is still written in Foundry as a standalone file.

### Rule 2: Real Contracts Only

Every contract the test interacts with must be a real deployed address, bound at the top of the file as a `constant`. Specifically:

- No mock contracts.
- No minimal reimplementations of protocol contracts.
- No stubbed interfaces that diverge from the deployed bytecode.
- Interfaces written in the test file must contain only the function signatures actually called, and those signatures must match the deployed contract exactly.

If a contract required by the attack path is not deployed on the target chain, stop. Do not fabricate a workaround. State the blocker to the user.

Addresses for test-only actors (attacker EOA, recipient) created via `makeAddr` are acceptable and expected. Addresses for protocol contracts, tokens, oracles, governance, and admins must be real.

### Rule 3: Fork Is Mandatory

The first statement inside `setUp()` must be `vm.createSelectFork(...)`. The fork uses the RPC for the correct chain and either the user-specified block or `latest`. State is read from the fork, not constructed locally.

### Rule 4: End-to-End Execution

The test must execute the full vulnerability path from the action that first triggers the vulnerable state to the final realized impact. This is as important as forking mainnet.

**Classify before writing.** Every finding falls into one of three categories, and the category determines where the test starts:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cholakovvv/foundry-poc-mainnet-fork](https://github.com/cholakovvv/foundry-poc-mainnet-fork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

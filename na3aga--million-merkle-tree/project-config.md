---
trigger: always_on
description: You are working on a Hardhat-based Merkle Tree Gas Analysis project that focuses on realistic, production-scale gas measurement for Merkle tree operations on Ethereum. The project uses an external data generation approach where trees are created off-chain in TypeScript and only verification operations are tested on-chain.
---

# Merkle Tree Gas Analysis Project - Cursor AI Rules

## Project Overview

You are working on a Hardhat-based Merkle Tree Gas Analysis project that focuses on realistic, production-scale gas measurement for Merkle tree operations on Ethereum. The project uses an external data generation approach where trees are created off-chain in TypeScript and only verification operations are tested on-chain.

### Core Technologies
- **Framework**: Hardhat (NOT Foundry) with TypeScript
- **Language**: TypeScript for scripts/tests, Solidity 0.8.24 for contracts
- **Libraries**: 
  - OpenZeppelin Contracts v5.1+ (on-chain verification)
  - OpenZeppelin Merkle Tree JS (off-chain generation)
  - hardhat-gas-reporter (gas analysis)
- **Testing**: Mocha/Chai with Hardhat's testing framework
- **Integration**: Context7 MCP for AI-assisted development

### Project Goals
1. Analyze gas costs for Merkle trees up to depth 20 (1,048,576 leaves)
2. Provide statistical analysis of gas scaling patterns
3. Test real-world integration scenarios (airdrops, voting, whitelists)
4. Create reusable patterns and benchmarks

## Development Standards

### TypeScript Standards

#### File Structure
```typescript
// Always use explicit imports
import { ethers } from "hardhat";
import { StandardMerkleTree } from "@openzeppelin/merkle-tree";
import { BigNumber } from "ethers";

// Group imports: external, internal, types
import type { MerkleVerifier } from "../typechain-types";
```

#### Naming Conventions
- Files: kebab-case (e.g., `generate-trees.ts`, `gas-analysis.test.ts`)
- Classes: PascalCase (e.g., `TreeGenerator`, `ProofSampler`)
- Functions: camelCase (e.g., `generateTreesForDepth`, `analyzeGasScaling`)
- Constants: UPPER_SNAKE_CASE (e.g., `MAX_TREE_DEPTH`, `DEFAULT_SAMPLE_SIZE`)

#### Type Safety
```typescript
// Always define interfaces for data structures
interface TreeData {
  depth: number;
  root: string;
  leafCount: number;
  sampleProofs: ProofData[];
  metadata: TreeMetadata;
}

// Use discriminated unions for variants
type AnalysisResult = 
  | { type: "success"; data: GasReport }
  | { type: "error"; error: string };

// Prefer readonly arrays and objects
function processProofs(proofs: readonly Proof[]): void {
  // Implementation
}
```

### Solidity Standards

#### Contract Structure
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {MerkleProof} from "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

/**
 * @title MerkleVerifier
 * @notice Gas-optimized Merkle proof verification
 * @dev Focuses on minimal gas consumption for analysis
 */
contract MerkleVerifier {
    // State variables
    mapping(bytes32 => bool) public roots;
    
    // Events
    event RootUpdated(bytes32 indexed root);
    
    // Custom errors (gas-efficient)
    error InvalidProof();
    error RootNotSet();
    
    // External functions
    function setRoot(bytes32 _root) external {
        roots[_root] = true;
        emit RootUpdated(_root);
    }
    
    // View functions
    function verify(
        bytes32[] calldata proof,
        bytes32 leaf
    ) external view returns (bool) {
        // Implementation
    }
}
```

#### Gas Optimization Patterns
```solidity
// Use calldata for read-only arrays
function verifyBatch(
    bytes32[][] calldata proofs,
    bytes32[] calldata leaves
) external view returns (bool[] memory results) {
    // Avoid storage reads in loops
    uint256 length = proofs.length;
    results = new bool[](length);
    
    // Cache frequently accessed values
    for (uint256 i; i < length;) {
        results[i] = _verify(proofs[i], leaves[i]);
        unchecked { ++i; }
    }
}
```

## Hardhat Integration Requirements

### Configuration Pattern
```typescript
// hardhat.config.ts
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import "hardhat-gas-reporter";

const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.24",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200, // Optimize for deployment
      },
      viaIR: true, // Enable IR optimizer for complex contracts
    },
  },
  gasReporter: {
    enabled: process.env.REPORT_GAS === "true",
    currency: "USD",
    gasPrice: 30,
    outputFile: "data/gas-reports/latest.txt",
    noColors: true,
  },
  networks: {
    hardhat: {
      blockGasLimit: 30_000_000, // Support large operations
    },
  },
};

export default config;
```

### Test Patterns
```typescript
// test/gas-analysis/verification-scaling.test.ts
import { expect } from "chai";
import { ethers } from "hardhat";
import { loadFixture } from "@nomicfoundation/hardhat-network-helpers";
import { loadTreeData } from "../../scripts/utils/DataPersistence";

describe("Verification Gas Scaling", function () {
  // Fixture pattern for consistent test setup
  async function deployVerifierFixture() {
    const [deployer] = await ethers.getSigners();
    const Verifier = await ethers.getContractFactory("MerkleVerifier");

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Na3aga) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

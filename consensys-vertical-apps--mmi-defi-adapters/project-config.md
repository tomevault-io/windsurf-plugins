---
trigger: always_on
description: DeFi Adapter Development Rules and Patterns
---


# DeFi Adapter Development Rules

This file contains key learnings and patterns for creating DeFi adapters, including both contract-based adapters (like Uniswap V4) and non-contract-based adapters (like ETH2 validator staking).

## 🚀 Quick Start Process

### 1. Initial Setup
```bash
# Always use nvm first
nvm use 22

# Create new adapter
npm run new-adapter

# Build contract types after adding ABIs
npm run build-types

# Run snapshots to test
npm run build-snapshots -- -p <protocol-name>
```

### 2. Essential Commands
- `npm run new-adapter` - Creates adapter structure
- `npm run build-types` - Generates TypeScript types from ABIs
- `npm run build-snapshots` - Tests adapter with real data
- `npm run build-snapshots -- -p <protocol>` - Test specific protocol

## 📁 File Structure

### Contract-Based Adapters
```
packages/adapters-library/src/adapters/<protocol>/
├── contracts/
│   ├── abis/
│   │   ├── ContractName.json          # ABI files
│   │   └── ...
│   ├── ContractName.ts                # Generated contract types
│   ├── index.ts                       # Export all contracts
│   └── factories/                     # Generated contract factories
├── products/
│   └── <product-type>/
│       ├── <protocol>Adapter.ts       # Main adapter implementation
│       ├── <protocol>-helper.ts       # Helper functions (optional)
│       └── tests/
│           └── testCases.ts           # Test cases with real addresses
└── ...
```

### Non-Contract-Based Adapters (External APIs)
```
packages/adapters-library/src/adapters/<protocol>/
├── products/
│   └── <product-type>/
│       ├── <protocol>Adapter.ts       # Main adapter implementation
│       └── tests/
│           ├── testCases.ts           # Test cases with real addresses
│           └── snapshots/
│               └── <chain>.positions.json  # Snapshot test results
└── ...
```

## 🔧 Key Implementation Patterns

### 1. Contract-Based Adapters

#### Contract Addresses Structure
```typescript
const contractAddresses: Partial<
  Record<Chain, { contract1: string; contract2: string }>
> = {
  [Chain.Ethereum]: {
    contract1: getAddress('0x...'),
    contract2: getAddress('0x...'),
  },
  // Add other chains as needed
}
```

#### Main Adapter Class Structure
```typescript
export class ProtocolAdapter implements IProtocolAdapter {
  productId = 'product-type'
  protocolId: Protocol
  chainId: Chain

  constructor(chainId: Chain) {
    this.chainId = chainId
    this.protocolId = Protocol.ProtocolName
  }

  async getPositions({
    userAddress,
    blockNumber,
    tokenIds,
  }: GetPositionsInput): Promise<ProtocolPosition[]> {
    // Implementation
  }

  async getTokenIds({
    userAddress,
    blockNumber,
  }: GetTokenIdsInput): Promise<string[]> {
    // Implementation
  }

  async unwrap({
    protocolTokenAddress,
    blockNumber,
  }: UnwrapInput): Promise<UnwrapExchangeRate> {
    throw new NotImplementedError()
  }
}
```

#### Position Processing Pattern
```typescript
return filterMapAsync(tokenIds, async (tokenId) => {
  try {
    // Fetch position data
    const result = await helperFunction(tokenId, contracts, blockNumber)
    
    if (!result) {
      return undefined
    }

    // Get token metadata
    const [token0Metadata, token1Metadata] = await Promise.all([
      this.helpers.getTokenMetadata(result.token0.address),
      this.helpers.getTokenMetadata(result.token1.address),
    ])

    // Create protocol token name
    const nftName = this.protocolTokenName(
      token0Metadata.symbol,
      token1Metadata.symbol,
      tokenId.toString(),
    )

    return {
      address: contractAddresses[this.chainId]!.mainContract,
      tokenId: tokenId.toString(),
      name: nftName,
      symbol: nftName,
      decimals: 18, // or appropriate value
      balanceRaw: BigInt(result.liquidity), // or appropriate balance
      type: TokenType.Protocol,
      tokens: [
        this.createUnderlyingToken(
          result.token0.address,
          token0Metadata,
          BigInt(result.token0.rawBalance),
          TokenType.UnderlyingClaimable,
        ),
        this.createUnderlyingToken(
          result.token1.address,
          token1Metadata,
          BigInt(result.token1.rawBalance),
          TokenType.UnderlyingClaimable,
        ),
      ],
    }
  } catch (error) {
    console.error('Error getting position:', error)
    return undefined
  }
})
```

### 2. Non-Contract-Based Adapters (External APIs)

#### External API Integration Pattern
```typescript
export class ProtocolAdapter implements IProtocolAdapter {
  productId = 'product-type'
  protocolId: Protocol
  chainId: Chain

  constructor(chainId: Chain) {
    this.chainId = chainId
    this.protocolId = Protocol.ProtocolName
  }

  async getPositions({
    tokenIds,
  }: GetPositionsInput): Promise<ProtocolPosition[]> {
    if (!tokenIds || tokenIds.length === 0) {
      return []
    }

    // Fetch data from external API
    const externalData = await this.fetchExternalData(tokenIds)

    if (!externalData) {
      return []
    }

    // Process and return positions
    const protocolTokens = await this.getProtocolTokens()
    const protocolToken = protocolTokens[0]!

    return [
      {
        address: protocolToken.address,
        name: protocolToken.name,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/consensys-vertical-apps) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

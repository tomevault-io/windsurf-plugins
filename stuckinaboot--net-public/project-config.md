---
trigger: always_on
description: This guide helps AI coding agents understand and contribute to the net-public repository effectively.
---

# AI Agent Guide for Net Protocol SDK

This guide helps AI coding agents understand and contribute to the net-public repository effectively.

## Quick Start

```bash
# Install dependencies
yarn install

# Build all packages
yarn build

# Run tests
yarn test

# Type check
yarn typecheck
```

## Repository Structure

```
net-public/
├── packages/               # SDK packages (npm published)
│   ├── net-core/           # Core messaging primitives (base package)
│   ├── net-storage/        # Onchain key-value storage
│   ├── net-netr/           # Memecoin-NFT token deployment
│   ├── net-feeds/          # Topic-based message feeds
│   ├── net-relay/          # Transaction relay service
│   └── net-cli/            # Command-line interface
├── examples/               # Example applications
│   └── basic-app/          # Next.js example with chat + storage
├── plugins/                # Claude Code plugin
│   └── net-protocol/       # AI assistance for Net development
└── docs/                   # Additional documentation
```

## Package Dependency Graph

```
@net-protocol/core          ← Base package (no internal deps)
       ↓
@net-protocol/storage       ← Depends on core
       ↓
@net-protocol/netr          ← Depends on core + storage

@net-protocol/feeds         ← Depends on core
@net-protocol/relay         ← Depends on core
@net-protocol/cli           ← Depends on core, storage, netr, relay
```

**Key insight:** Changes to `core` may affect all other packages. Always run full test suite after modifying core.

## Contract Addresses

All contracts use the same address across all supported EVM chains:

| Contract | Address |
|----------|---------|
| Net (Messaging) | `0x00000000B24D62781dB359b07880a105cD0b64e6` |
| Storage | `0x00000000DB40fcB9f4466330982372e27Fd7Bbf5` |
| Netr/Banger | Chain-specific (see `packages/net-netr/src/chainConfig.ts`) |

## Supported Chains

**Messages & Storage:** Base (8453), Ethereum (1), Degen (666666666), Ham (5112), Ink (57073), Unichain (130), HyperEVM (999), Plasma (9745), Monad (143), Base Sepolia (84532), Sepolia (11155111)

**Token Deployment (Netr):** Base (8453), Plasma (9745), Monad (143), HyperEVM (999)

## Code Patterns

### Package Structure

Each package follows this structure:

```
packages/net-{name}/
├── src/
│   ├── index.ts           # Public exports (IMPORTANT: all public API here)
│   ├── types.ts           # TypeScript types
│   ├── constants.ts       # ABIs, addresses, config values
│   ├── chainConfig.ts     # Chain-specific configuration
│   ├── client/            # Non-React client classes
│   │   ├── index.ts       # Re-exports
│   │   └── {Name}Client.ts
│   ├── hooks/             # React hooks (wagmi-based)
│   │   ├── index.ts       # Re-exports
│   │   └── use{Feature}.ts
│   ├── utils/             # Utility functions
│   └── __tests__/         # Tests (vitest)
├── package.json
├── tsconfig.json
└── README.md
```

### React Hooks Pattern

Hooks use wagmi's `useReadContract` and return `{ data, isLoading, error }`:

```typescript
import { useReadContract } from "wagmi";
import { useMemo } from "react";

export function useNetMessages(params: UseNetMessagesOptions) {
  const readContractArgs = useMemo(
    () => getNetMessagesReadConfig({
      chainId: params.chainId,
      filter: params.filter,
    }),
    [params.chainId, params.filter]
  );

  const { data, isLoading, error } = useReadContract({
    ...readContractArgs,
    query: { enabled: params.enabled },
  });

  const messages = useMemo(() => (data as NetMessage[]) ?? [], [data]);

  return { messages, isLoading, error: error as Error | undefined };
}
```

### Client Pattern

Clients use viem's `PublicClient` for read operations:

```typescript
import { getPublicClient } from "../chainConfig";

export class NetClient {
  private chainId: number;
  private publicClient: PublicClient;

  constructor(options: NetClientOptions) {
    this.chainId = options.chainId;
    this.publicClient = getPublicClient({
      chainId: options.chainId,
      rpcUrl: options.overrides?.rpcUrls,
    });
  }

  async getMessages(params: GetMessagesParams): Promise<NetMessage[]> {
    // Use this.publicClient.readContract(...)
  }
}
```

### Chain Configuration Pattern

Each package that needs chain config has a `chainConfig.ts`:

```typescript
const CHAIN_CONFIG: Record<number, ChainConfig> = {
  8453: {  // Base
    name: "Base",
    rpcUrls: ["https://base-mainnet.public.blastapi.io"],
    contractAddress: "0x...",
  },
  // ... more chains
};

export function getChainRpcUrls(params: { chainId: number; rpcUrl?: string }): string[] {
  // Priority: per-call override > global override > defaults
}
```

### CLI Command Pattern

Commands use the `commander` library:

```typescript
// src/commands/{name}/index.ts
import { Command } from "commander";
import { parseCommonOptions } from "../../cli/shared";

export function registerMyCommand(program: Command): void {
  const myCommand = new Command("mycommand")
    .description("Description here")
    .requiredOption("--param <value>", "Parameter description")
    .option("--chain-id <id>", "Chain ID")
    .option("--private-key <key>", "Private key")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stuckinaboot/net-public](https://github.com/stuckinaboot/net-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

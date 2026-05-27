---
trigger: always_on
description: In version 2.0, useConnex is replaced with useThor.
---


# VeChain Kit Version 2 Migration Guide

In version 2.0, useConnex is replaced with useThor.

To migrate connex with thor client, use contract call patterns below.

## Single Contract Call Pattern (useCallClause)

When reading data from a single contract method, follow this pattern:

### 1. Import Required Dependencies

```typescript
import { getConfig } from '@/config';
import { NETWORK_TYPE } from '@/config/network';
import { ContractName__factory } from '@/contracts';
import { useVeChainKitConfig } from '@/providers';
import { getCallClauseQueryKeyWithArgs, useCallClause } from '@/hooks';
```

### 2. Define ABI and Method

```typescript
const abi = ContractName__factory.abi;
const method = 'methodName' as const;
```

### 3. Create Query Key Function

```typescript
export const getContractDataQueryKey = (
    param1: string,
    networkType: NETWORK_TYPE,
) =>
    getCallClauseQueryKeyWithArgs({
        abi,
        address: getConfig(networkType).contractAddress as `0x${string}`,
        method,
        args: [BigInt(param1 ?? 0)], // convert args to required params by TS
    });
```

### 4. Create Hook with useCallClause

```typescript
export const useContractData = (param1?: string) => {
    const { network } = useVeChainKitConfig();

    const address = getConfig(network.type).contractAddress as `0x${string}`;

    return useCallClause({
        abi,
        address,
        method,
        args: [BigInt(param1 ?? 0)],
        queryOptions: {
            enabled: !!param1, // Enable query conditionally
            select: (data) =>
                // Example of  transforming raw contract data,
                data[0].map((item) => ({
                    id: item.id.toString(),
                    // Convert BigInt values to strings
                    timestamp: item.timestamp.toString(),
                    // Keep other fields as needed
                    name: item.name,
                })),
        },
    });
};
```

## Multiple Contract Calls Pattern (executeMultipleClausesCall)

When you need to make multiple contract calls in parallel, use this pattern:

### 1. Import Required Dependencies

```typescript
import { useQuery } from '@tanstack/react-query';
import { ContractName__factory } from '@/contracts';
import { getConfig } from '@/config';
import { useThor } from '@vechain/dapp-kit-react';
import { useVeChainKitConfig } from '@/providers';
import { executeMultipleClausesCall } from '@/utils';
```

### 2. Define Query Key Function

```typescript
export const getMultipleDataQueryKey = (param?: number | string) => [
    'VECHAIN_KIT',
    'CONTRACT_NAME',
    'METHOD_NAME',
    param,
];
```

### 3. Create Hook with executeMultipleClausesCall

```typescript
export const useMultipleContractData = (items: string[], roundId?: string) => {
    const thor = useThor();
    const { network } = useVeChainKitConfig();

    const address = getConfig(network.type).contractAddress as `0x${string}`;

    return useQuery({
        queryKey: getMultipleDataQueryKey(roundId),
        queryFn: async () => {
            const results = await executeMultipleClausesCall({
                thor,
                calls: items.map(
                    (item) =>
                        ({
                            abi: ContractName__factory.abi,
                            functionName: 'methodName',
                            address,
                            args: [roundId, item],
                        } as const),
                ),
            });

            // Process and transform results
            return results.map((result, index) => ({
                item: items[index] as string,
                value: Number(result[0] || 0),
                // Transform additional fields as needed
                secondaryValue: Number(result[1] || 0) / 100, // Convert basis points to percentage
            }));
        },
        enabled: !!roundId && !!items.length, // Enable when both dependencies are available
    });
};
```

## Transaction Building Pattern (useBuildTransaction)

When building and sending transactions, use this pattern:

### 1. Import Required Dependencies

```typescript
import {
    useWallet,
    useThor,
    useBuildTransaction,
    useTransactionModal,
    useTransactionToast,
    TransactionModal,
    TransactionToast,
} from '@vechain/vechain-kit';
import { ContractName__factory } from '@vechain/vechain-kit/contracts';
```

### 2. Set up Transaction Hook

```typescript
const { account } = useWallet();
const thor = useThor();

const {
    sendTransaction,
    status,
    txReceipt,
    isTransactionPending,
    error,
    resetStatus,
} = useBuildTransaction({
    clauseBuilder: () => {
        if (!account?.address) return [];

        return [
            {
                // example of loading clause using ThorClient
                ...thor.contracts
                    .load(contractAddress, ContractName__factory.abi)
                    .clause.methodName(param1, param2).clause,
                comment: `Description of what this transaction does`,
            },
        ];
    },
});
```

## Key Patterns and Best Practices

### 2. Contract args


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vechain/vechain-kit](https://github.com/vechain/vechain-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->

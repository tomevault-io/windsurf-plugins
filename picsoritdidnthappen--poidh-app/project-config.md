---
trigger: always_on
description: Post bounties and evaluate/accept winning submissions on poidh (pics or it didn't happen) on Arbitrum, Base, or Degen Chain. Use this skill when the user wants to create a bounty on poidh.xyz, post a task with an ETH or DEGEN reward on-chain, evaluate photo submissions using vision, accept a winning claim on a solo bounty, or initiate/resolve voting on an open bounty.
---


## Overview

This skill interacts with the PoidhV3 contracts on Arbitrum, Base, and Degen Chain to:

1. **Post bounties** (solo or open)
2. **Evaluate claim submissions** using vision — fetch the image URI from each claim and compare against the bounty description
3. **Accept the winning claim** (solo bounty) or **initiate + resolve a vote** (open bounty)

**poidh** ("pics or it didn't happen") is a fully on-chain bounty protocol. Claimants submit photo proof, and the bounty issuer (or contributors via vote) accepts the best claim to release funds.

> ⚠️ The PoidhV3 contract enforces `msg.sender == tx.origin`. Only **EOA wallets** can create or accept bounties. Smart contract wallets (Safe, etc.) will revert with `ContractsCannotCreateBounties`.

---

## Required Environment Variables

| Variable      | Description                                                                  |
| ------------- | ---------------------------------------------------------------------------- |
| `PRIVATE_KEY` | Private key of the EOA signing transactions (hex, with or without 0x prefix) |
| `RPC_URL`     | RPC URL for the target chain                                                 |
| `POIDH_CHAIN` | Target chain: `arbitrum`, `base`, or `degen`                                 |

`POIDH_CONTRACT_ADDRESS` is resolved automatically from `POIDH_CHAIN` — do not set it manually.

---

## Supported Chains

| Chain       | Contract Address                             | Explorer            |
| ----------- | -------------------------------------------- | ------------------- |
| Arbitrum    | `0x5555Fa783936C260f77385b4E153B9725feF1719` | arbiscan.io         |
| Base        | `0x5555Fa783936C260f77385b4E153B9725feF1719` | basescan.org        |
| Degen Chain | `0x18E5585ca7cE31b90Bc8BB7aAf84152857cE243f` | explorer.degen.tips |

> ⚠️ Minimum amounts differ by chain. On **Arbitrum and Base**: `0.001 ETH` minimum bounty, `0.00001 ETH` minimum contribution. On **Degen Chain**: `1000 DEGEN` minimum bounty, `10 DEGEN` minimum contribution. Always verify on-chain before posting:
>
> ```bash
> cast call $POIDH_CONTRACT_ADDRESS "MIN_BOUNTY_AMOUNT()(uint256)" --rpc-url $RPC_URL
> cast call $POIDH_CONTRACT_ADDRESS "MIN_CONTRIBUTION()(uint256)" --rpc-url $RPC_URL
> ```

Resolve the contract address at the start of every session:

```bash
if [ "$POIDH_CHAIN" = "degen" ]; then
  POIDH_CONTRACT_ADDRESS="0x18E5585ca7cE31b90Bc8BB7aAf84152857cE243f"
else
  # arbitrum and base share the same address
  POIDH_CONTRACT_ADDRESS="0x5555Fa783936C260f77385b4E153B9725feF1719"
fi
```

The poidh.xyz URL also changes per chain:

```bash
if [ "$POIDH_CHAIN" = "arbitrum" ]; then
  POIDH_BASE_URL="https://poidh.xyz/arbitrum"
  POIDH_V2_OFFSET=180
elif [ "$POIDH_CHAIN" = "degen" ]; then
  POIDH_BASE_URL="https://poidh.xyz/degen"
  POIDH_V2_OFFSET=1197
else
  POIDH_BASE_URL="https://poidh.xyz/base"
  POIDH_V2_OFFSET=986
fi
```

---

## Part 1: Posting a Bounty

### Check Minimum Bounty Amount

```bash
cast call $POIDH_CONTRACT_ADDRESS "MIN_BOUNTY_AMOUNT()(uint256)" --rpc-url $RPC_URL
```

### Post a Solo Bounty

Solo = only you fund it; you accept claims directly with no vote required.

```bash
cast send $POIDH_CONTRACT_ADDRESS \
  "createSoloBounty(string,string)" \
  "<BOUNTY_NAME>" \
  "<BOUNTY_DESCRIPTION>" \
  --value <AMOUNT> \
  --private-key $PRIVATE_KEY \
  --rpc-url $RPC_URL
```

**Example:**

```bash
cast send $POIDH_CONTRACT_ADDRESS \
  "createSoloBounty(string,string)" \
  "Brooklyn Bridge at sunset" \
  "High quality photo of the Brooklyn Bridge during golden hour. Must show the full span." \
  --value 0.001ether \
  --private-key $PRIVATE_KEY \
  --rpc-url $RPC_URL
```

### Post an Open Bounty

Open = others can co-fund; claim acceptance requires a contributor-weighted vote.

```bash
cast send $POIDH_CONTRACT_ADDRESS \
  "createOpenBounty(string,string)" \
  "<BOUNTY_NAME>" \
  "<BOUNTY_DESCRIPTION>" \
  --value <AMOUNT> \
  --private-key $PRIVATE_KEY \
  --rpc-url $RPC_URL
```

### Get the Bounty ID After Posting

```bash
cast receipt <TX_HASH> --rpc-url $RPC_URL --json | \
  python3 -c "
import sys, json
receipt = json.load(sys.stdin)
for log in receipt['logs']:
    if log['address'].lower() == '${POIDH_CONTRACT_ADDRESS}'.lower() and len(log['topics']) >= 2:
        bounty_id = int(log['topics'][1], 16)
        frontend_id = bounty_id + ${POIDH_V2_OFFSET}
        print(f'Bounty ID: {bounty_id}')
        print(f'View at: ${POIDH_BASE_URL}/bounty/{frontend_id}')
        break
"
```

---

## Part 2: Evaluating Claims

When the user wants to pick a winner, the agent must:

1. Fetch all claims for the bounty
2. Retrieve each claim's URI from the NFT contract
3. Fetch and evaluate the content against the bounty description
4. Pick the best match

Claim submissions are freeform — the URI could point to an image, a video, a tweet, a GitHub PR, a webpage, a document, or anything else. Evaluate whatever you find against what the bounty asked for.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [picsoritdidnthappen/poidh-app](https://github.com/picsoritdidnthappen/poidh-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

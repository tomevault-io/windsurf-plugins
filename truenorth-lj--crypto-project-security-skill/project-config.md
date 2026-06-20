---
trigger: always_on
description: Analyze a DeFi protocol's risk profile across smart contract, off-chain, and track-record dimensions. Use when the user wants a risk analysis of a DeFi project, to check protocol security, or to assess risk. Trigger words include "risk analysis", "analyze protocol", "audit defi", "check security", "defi risk", "protocol vulnerability", "is it safe".
---


# DeFi Risk Analysis Skill

Perform a comprehensive risk analysis of a DeFi protocol. This skill systematically evaluates smart contract risk, off-chain risk (governance, team, operations), and track record (historical incidents, battle-testing, response capability).

This is a structured **risk analysis**, NOT a formal smart contract audit — it reviews publicly available information and on-chain state to surface risk signals. Formal audits require line-by-line code review by professional auditing firms.

## Input

The user provides one or more of:
- Protocol name (e.g., "Aave", "Drift", "GMX")
- Protocol website or DeFiLlama URL
- Contract addresses or chain

## Workflow

### Step 0: Quick Triage (Red Flag Scan)

Before deep analysis, run a quick triage to decide analysis priority:

1. **DeFiLlama data check**:
   First, resolve the protocol name to the correct DeFiLlama slug (slugs are non-obvious, e.g., "maker" not "sky", "pancakeswap" not "pancake-swap"):
   ```bash
   # Fetch all protocols and fuzzy-match by name
   curl -s 'https://api.llama.fi/protocols' | jq -r '.[] | select(.name | test("{protocol}"; "i")) | "\(.slug) -- \(.name) -- TVL: \(.tvl)"'
   ```
   If no match, try partial name or check the protocol's website for its DeFiLlama listing.
   Then fetch full data with the resolved slug: `curl -s 'https://api.llama.fi/protocol/{slug}'` to get:
   - Current TVL and TVL history (sharp drops = red flag)
   - Number of audits listed
   - Chain(s)
2. **GoPlus token security check**: If the protocol has a governance/utility token on an EVM chain, run `./scripts/goplus-check.sh token <chain_id> <contract_address>` or call the API directly:
   ```bash
   curl -s "https://api.gopluslabs.io/api/v1/token_security/<chain_id>?contract_addresses=<address>"
   ```
   Extract these red flags from the response:
   - `is_honeypot = 1` -- token is a honeypot (CRITICAL)
   - `honeypot_with_same_creator = 1` -- creator has deployed honeypots (CRITICAL)
   - `is_open_source = 0` -- contract not verified (HIGH)
   - `hidden_owner = 1` -- hidden ownership mechanism (HIGH)
   - `owner_change_balance = 1` -- owner can modify balances (HIGH)
   - `selfdestruct = 1` -- contract can self-destruct (HIGH)
   - `can_take_back_ownership = 1` -- can reclaim ownership after renouncing (HIGH)
   - `is_proxy = 1` -- upgradeable proxy (MEDIUM, cross-reference with Step 2)
   - `is_mintable = 1` -- unlimited minting possible (MEDIUM)
   - `slippage_modifiable = 1` -- owner can change tax/slippage (MEDIUM)
   - `transfer_pausable = 1` -- transfers can be paused (MEDIUM)
   - `is_blacklisted = 1` -- has blacklist functionality (MEDIUM)

   Also note: `buy_tax`, `sell_tax`, `holder_count`, `lp_holders` (lock status), and `trust_list` status.

   **Chain IDs**: 1=Ethereum, 56=BSC, 137=Polygon, 42161=Arbitrum, 10=Optimism, 43114=Avalanche, 8453=Base, 324=zkSync. Solana is NOT supported by GoPlus token security API.

   **Solana token fallback**: GoPlus does not support Solana SPL tokens. For Solana protocols, use these alternatives instead:
   - **RugCheck**: `curl -s 'https://api.rugcheck.xyz/v1/tokens/{mint_address}/report'` -- returns risk score, mutable metadata, freeze authority, mint authority, top holders, LP lock status
   - **Birdeye**: `curl -s -H 'X-API-KEY: public' 'https://public-api.birdeye.so/public/token_security?address={mint_address}'` -- holder concentration, LP info
   - **Manual checks**: On Solana Explorer, verify: (1) mint authority (revoked = safer), (2) freeze authority (revoked = safer), (3) metadata mutability, (4) top holder concentration
   Record the source as "RugCheck" or "Birdeye" instead of "GoPlus" in the report. If none of the alternatives return data, record "Solana Token Check: UNAVAILABLE" and note the gap.

   **Error handling**: GoPlus is a free API with undocumented rate limits. If the API returns an error, empty result, or times out:
   - Record "GoPlus: UNAVAILABLE" in the report rather than omitting the section
   - Wait 5 seconds and retry once
   - If still failing, proceed with the analysis without GoPlus data and note the gap in Information Gaps

3. **GoPlus address check** (optional): If specific admin/deployer addresses are known, check for malicious history:
   ```bash
   curl -s "https://api.gopluslabs.io/api/v1/address_security/<address>?chain_id=<chain_id>"
   ```
   Flags: `cybercrime`, `money_laundering`, `phishing_activities`, `stealing_attack`, `sanctioned`, `honeypot_related_address`, `malicious_mining_activities`, `number_of_malicious_contracts_created`.

4. **Immediate red flags** (any = escalate to CRITICAL triage):
   - TVL = $0 or dropped >50% in 30 days
   - No audits listed on DeFiLlama
   - Protocol age < 6 months with TVL > $50M
   - Anonymous team with no prior track record
   - Closed-source contracts
   - GoPlus: honeypot detected or creator has honeypot history
   - GoPlus: hidden owner or owner can change balances
   - GoPlus: admin/deployer address flagged as malicious

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [truenorth-lj/crypto-project-security-skill](https://github.com/truenorth-lj/crypto-project-security-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

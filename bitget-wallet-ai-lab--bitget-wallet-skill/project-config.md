---
trigger: always_on
description: Interact with Bitget Wallet API for crypto market data, token info, swap quotes, RWA (real-world asset) stock trading, and security audits. Use when the user asks about wallet, token prices, market data, swap/trading quotes, RWA stock discovery and trading, token security checks, K-line charts, or token rankings on supported chains (ETH, SOL, BSC, Base, etc.).
---


# Bitget Wallet Skill

## API Overview

**⚠️ MANDATORY: Load Domain Knowledge Before Any API Call**

**Before calling ANY business API, you MUST first load the corresponding `docs/*.md` file for that domain.** This is non-negotiable — domain knowledge contains critical flow rules, parameter constraints, and pitfalls that cannot be inferred from command syntax alone. Skipping this step leads to silent failures, incorrect parameters, or broken flows.

| Business Domain | Must Load First | Before Calling |
|----------------|----------------|----------------|
| Swap / Trade | [`docs/swap.md`](docs/swap.md) | quote, confirm, make-order, send, get-order-details |
| Market Data / Token Analysis | [`docs/market-data.md`](docs/market-data.md) | coin-market-info, security, coin-dev, kline, tx-info, liquidity, rankings, launchpad-tokens, search-tokens-v3 |
| Alpha Intelligence | [`docs/alpha.md`](docs/alpha.md) | alpha-gems, alpha-signals, alpha-hunter-find, alpha-hunter-detail, agent-alpha-tags, agent-alpha-hunter-find, multi-agent-signal |
| Token Deep Analysis | [`docs/token-analyze.md`](docs/token-analyze.md) | simple-kline, trading-dynamics, transaction-list, holders-info, profit-address-analysis, top-profit, compare-tokens |
| Address Discovery | [`docs/address-find.md`](docs/address-find.md) | recommend-address-list |
| Wallet / Signing | [`docs/wallet-signing.md`](docs/wallet-signing.md) | Any signing operation, key derivation, order_sign.py, order_make_sign_send.py |
| Social Login Wallet | [`docs/social-wallet.md`](docs/social-wallet.md) | social-wallet.py sign_transaction, sign_message, get_address |
| RWA Stock Trading | [`docs/rwa.md`](docs/rwa.md) | Any RWA stock discovery, config, order, holdings |
| x402 Payments | [`docs/x402-payments.md`](docs/x402-payments.md) | x402_pay.py, HTTP 402 payment flow |
| Token Transfer / Gasless | [`docs/transfer.md`](docs/transfer.md) | transfer_make_sign_send.py, social_transfer_make_sign_send.py, get-transfer-order |
| First-Time Setup | [`docs/first-time-setup.md`](docs/first-time-setup.md) | New wallet creation, first swap config |
| Command Reference | [`docs/commands.md`](docs/commands.md) | When unsure about subcommand parameters or usage |

**Do NOT rely solely on the Quick Reference or examples in this SKILL.md — they are summaries only.** The `docs/` files are the authoritative source for each domain's rules and flow.

---

**How to handle tasks:**

1. **Load domain knowledge first (mandatory):** Read the relevant `docs/*.md` file(s) from the table above before making any API calls for that domain.
2. **Primary sources:** Use the **Scripts** section in this SKILL and the files under **`docs/`** to decide which commands to run and how. Scripts lists each Python CLI with purpose, subcommands, and when to use them; `docs/swap.md`, `docs/wallet-signing.md`, `docs/market-data.md`, etc. describe flows and domain rules.
3. **Run commands as documented:** Execute the script invocations shown in Scripts (e.g. `python3 scripts/bitget-wallet-agent-api.py ...`, `python3 scripts/order_sign.py ...`). For swap, balance, wallet, and signing, follow the flows in `docs/swap.md` and `docs/wallet-signing.md`.

**Balance query — choose the right API for the task:**

**Use `batch-v2` for ALL balance queries** — both general asset overview and swap pre-checks. It returns balance + price + token info in one call and supports all chains including Tron.

**Before starting a new swap - two mandatory pre-checks:**

1. **Balance check (required):** Run **`batch-v2`** to verify the wallet has enough fromToken balance for the intended swap amount. Include native token (`""`) to check gas availability. If `fromToken balance < fromAmount`, inform the user of the shortfall and **do not proceed**. **Gas mode decision:** If native token balance is sufficient for gas → use `--feature user_gas` (preferred). If native token balance is near zero → use `--feature no_gas` (gasless, gas deducted from fromToken; **requires swap amount ≥ ~$5 USD** — below this threshold the API only returns `user_gas`). This choice must be passed to confirm.
   ```bash
   python3 scripts/bitget-wallet-agent-api.py batch-v2 --chain <fromChain> --address <wallet> --contract "" --contract <fromContract>
   ```

2. **Token risk check (required):** Run **`check-swap-token`** for the intended fromToken and toToken. If `error_code != 0`, show `msg` and stop. If for any token `data.list[].checkTokenList` is non-empty, show the `tips` content to the user and let them decide whether to continue. If the **toToken** (swap target) has an item with **`waringType` equal to `"forbidden-buy"`**, do **not** proceed with the swap and warn the user that this token cannot be used as the swap target.
   ```bash
   python3 scripts/bitget-wallet-agent-api.py check-swap-token --from-chain ... --from-contract ... --from-symbol ... --to-chain ... --to-contract ... --to-symbol ...
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitget-wallet-ai-lab/bitget-wallet-skill](https://github.com/bitget-wallet-ai-lab/bitget-wallet-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

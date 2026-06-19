---
trigger: always_on
description: This skill uses date-based versioning (`YYYY.M.DD`). Each release includes a sequential suffix: `YYYY.M.DD-1`, `YYYY.M.DD-2`, etc. The current version is in the frontmatter above. See `CHANGELOG.md` for full history.
---

---
name: bitget-wallet
version: "2026.3.12-1"
updated: "2026-03-12"
description: "Interact with Bitget Wallet API for crypto market data, token info, swap quotes, RWA (real-world asset) stock trading, and security audits. Use when the user asks about wallet, token prices, market data, swap/trading quotes, RWA stock discovery and trading, token security checks, K-line charts, or token rankings on supported chains (ETH, SOL, BSC, Base, etc.)."
---

# Bitget Wallet Skill

## API Overview

**How to handle tasks:**

1. **Primary sources:** Use the **Scripts** section in this SKILL and the files under **`docs/`** to decide which commands to run and how. Scripts lists each Python CLI with purpose, subcommands, and when to use them; `docs/swap.md`, `docs/wallet-signing.md`, `docs/market-data.md`, etc. describe flows and domain rules.
2. **Run commands as documented:** Execute the script invocations shown in Scripts (e.g. `python3 scripts/bitget_agent_api.py ...`, `python3 scripts/order_sign.py ...`). For swap, balance, wallet, and signing, follow the flows in `docs/swap.md` and `docs/wallet-signing.md`.

**Before starting a new swap - two mandatory pre-checks:**

1. **Balance check (required):** Run **`get-processed-balance`** to verify the wallet has enough fromToken balance for the intended swap amount. Include native token (`""`) to check gas availability. If `fromToken balance < fromAmount`, inform the user of the shortfall and **do not proceed**. **Gas mode decision:** If native token balance is sufficient for gas → use `--feature user_gas` (preferred). If native token balance is near zero → use `--feature no_gas` (gasless, gas deducted from fromToken; **requires swap amount ≥ ~$5 USD** — below this threshold the API only returns `user_gas`). This choice must be passed to confirm.
   ```bash
   python3 scripts/bitget_agent_api.py get-processed-balance --chain <fromChain> --address <wallet> --contract "" --contract <fromContract>
   ```

2. **Token risk check (required):** Run **`check-swap-token`** for the intended fromToken and toToken. If `error_code != 0`, show `msg` and stop. If for any token `data.list[].checkTokenList` is non-empty, show the `tips` content to the user and let them decide whether to continue. If the **toToken** (swap target) has an item with **`waringType` equal to `"forbidden-buy"`**, do **not** proceed with the swap and warn the user that this token cannot be used as the swap target.
   ```bash
   python3 scripts/bitget_agent_api.py check-swap-token --from-chain ... --from-contract ... --from-symbol ... --to-chain ... --to-contract ... --to-symbol ...
   ```

**Swap execution must strictly follow `docs/swap.md` Flow Overview — no shortcuts:**

1. **Balance check** — verify fromToken + native token balance before anything else
2. **Token risk check** — check-swap-token for both fromToken and toToken
3. **Quote** — display **all** market results to user, recommend the first, let user choose
4. **Confirm** — must display three fields to user: `outAmount` (expected), `minAmount` (minimum), `gasTotalAmount` (gas cost); check `recommendFeatures` for gas sufficiency
5. **User confirmation** — **do not** sign or send until user explicitly confirms ("confirm", "execute", "yes")
6. **makeOrder + sign + send** — execute as one atomic operation (use `order_make_sign_send.py`)
7. **Query status** — check order result; ignore `tips` when status=success

See Scripts for full command details and `docs/swap.md` for the complete flow.

**Technical reference (no need to read .py files):**

- **Base URL:** `https://copenapi.bgwapi.io` (token auth, no API key needed).
- **Agent API:** Invoked via `scripts/bitget_agent_api.py`. Covers token risk check (`check-swap-token`), balance (`get-processed-balance`), balance+price (`batch-v2`), token search (`search-tokens`), token list (`get-token-list`); **market data:** token-info, token-price, batch-token-info, kline, tx-info, batch-tx-info, historical-coins, rankings, liquidity, security; **RWA:** rwa-get-user-ticker-selector, rwa-get-config, rwa-stock-info (GET), rwa-stock-order-price, rwa-kline, rwa-get-my-holdings; swap flow (quote, confirm, make-order, send, get-order-details). Request/response details are in `docs/`.
- **Requests:** POST with JSON body; RWA stock info uses GET with query params.

## Domain Knowledge

### Skill Domain Knowledge

#### Version Awareness

This skill uses date-based versioning (`YYYY.M.DD`). Each release includes a sequential suffix: `YYYY.M.DD-1`, `YYYY.M.DD-2`, etc. The current version is in the frontmatter above. See `CHANGELOG.md` for full history.

**First-use version check (at most once every 7 days):**

1. Check the installed `version` from frontmatter above.
2. Fetch the latest CHANGELOG from `https://raw.githubusercontent.com/bitget-wallet-ai-lab/bitget-wallet-skill/main/CHANGELOG.md`.
3. Compare with the installed version.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pitrian/solana-guardian-ai-agent](https://github.com/pitrian/solana-guardian-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

---
trigger: always_on
description: Connect to EmblemVault and manage crypto wallets via EmblemAI. Supports Solana, Ethereum, Base, BSC, Polygon, Hedera, and Bitcoin. Use when the user wants to trade crypto, check balances, swap tokens, interact with blockchain wallets, or create and use profile-scoped agent wallets.
---


# Emblem Agent Wallet

Connect to **EmblemAI** -- EmblemVault's autonomous crypto AI with 250+ trading tools across 7 blockchains. Browser auth, streaming responses, profile-scoped local state, plugin system, and zero-config agent mode.

**Requires the CLI**: `npm install -g @emblemvault/agentwallet`

---

## Quick Start -- How to Use This Skill

**Step 1: Install the CLI**

```bash
npm install -g @emblemvault/agentwallet
```

This provides a single command: `emblemai`

**Step 2: Use it**

When this skill loads, you can ask EmblemAI anything about crypto:

- "What are my wallet addresses?"
- "Show my balances across all chains"
- "What's trending on Solana?"
- "Swap $20 of SOL to USDC"
- "Send 0.1 ETH to 0x..."

For zero-config agent provisioning, a profile can create and persist its own wallet with a single command:

```bash
emblemai --agent --profile hustle -m "What are my wallet addresses?"
```

**To invoke this skill, say things like:**
- "Use my Emblem wallet to check balances"
- "Ask EmblemAI what tokens I have"
- "Connect to EmblemVault"
- "Check my crypto portfolio"
- "Create or use my agent wallet profile and show my addresses"

All requests are routed through `emblemai` under the hood. If more than one profile exists, every agent-mode invocation must include `--profile <name>`.

---

## Prerequisites

- **Node.js** >= 20.18.0
- **Terminal** with 256-color support (iTerm2, Kitty, Windows Terminal, or any xterm-compatible terminal)
- **Optional**: [glow](https://github.com/charmbracelet/glow) for rich markdown rendering (`brew install glow` on macOS)

## Installation

### From npm (Recommended)

```bash
npm install -g @emblemvault/agentwallet
```

### From source

```bash
git clone https://github.com/EmblemCompany/EmblemAi-cli.git
cd EmblemAi-cli
npm install
npm link   # makes `emblemai` available globally
```

## First Run

1. Install: `npm install -g @emblemvault/agentwallet`
2. Create or pick a profile: `emblemai profile create hustle`
3. Run either `emblemai --profile hustle` or `emblemai --agent --profile hustle -m "What are my wallet addresses?"`
4. Check `/plugins` to see which plugins loaded
5. Type `/help` to see all commands
6. Back up profile auth immediately after first wallet creation

---

## Authentication

EmblemAI v3 supports two authentication methods with profile-scoped local state: **browser auth** for interactive use and **password auth** for agent/scripted use.

### Profile Rules

Profiles are the canonical multi-agent isolation mechanism.

- `emblemai profile list`
- `emblemai profile create <name>`
- `emblemai profile use <name>`
- `emblemai profile inspect [name]`
- `emblemai profile delete <name>`
- `emblemai --profile <name> ...`

**Fail closed rule:** if more than one profile exists in `~/.emblemai`, every `--agent` invocation must include `--profile <name>`. Agent mode never guesses which wallet identity to use.

Using separate `HOME` directories is optional isolation, not the primary pattern. Prefer profiles first.

### Browser Auth (Interactive Mode)

When you run `emblemai --profile <name>` without `-p`, the CLI:

1. Checks the selected profile for a saved session
2. If a valid (non-expired) session exists, restores it instantly -- no login needed
3. If no session, starts a local server on `127.0.0.1:18247` and opens your browser
4. You authenticate via the EmblemVault auth modal in the browser
5. The session JWT is captured, saved to that profile on disk, and the CLI proceeds
6. If the browser can't open, the URL is printed for manual copy-paste
7. If authentication times out (5 minutes), falls back to a password prompt for the same profile

### Password Auth (Agent Mode)

**Login and signup are the same action.** The first use of a password creates a vault; subsequent uses of the same password return the same vault. Different passwords produce different wallets, while profiles isolate the local session and stored credentials around those wallets.

In agent mode, if no password is provided for the selected profile, a secure random password is auto-generated and stored encrypted via dotenvx under `~/.emblemai/profiles/<name>/`. Agent mode works out of the box with no manual setup.

### What Happens on Authentication

1. The selected profile is resolved
2. Browser auth: session JWT is received from browser and hydrated into the SDK
   Password auth: password is sent to `EmblemAuthSDK.authenticatePassword()` and can be stored in the selected profile for reuse
3. A deterministic vault is derived -- same credentials always yield the same vault
4. The session provides wallet addresses across multiple chains: Solana, Ethereum, Base, BSC, Polygon, Hedera, Bitcoin
5. The EmblemAI client is initialized with the profile's session

### Credential Discovery

Before making requests for a profile, locate the password using this priority:

| Method | How to use | Priority |
|--------|-----------|----------|
| CLI argument | `emblemai --profile <name> -p "your-password"` | 1 (highest, stored encrypted) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EmblemCompany/EmblemAi-AgentWallet](https://github.com/EmblemCompany/EmblemAi-AgentWallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

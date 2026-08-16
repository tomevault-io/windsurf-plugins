---
trigger: always_on
description: **Apex is Kaggle for decentralized intelligence.** The difference: Kaggle captures
---

# Apex - Bittensor Subnet 1

**Apex is Kaggle for decentralized intelligence.** The difference: Kaggle captures
*predictions*, while Apex captures *intelligence* — the actual models and algorithms
that produce them. It runs open competitions where
anyone can submit an algorithm or model to solve hard problems, scores every entry
automatically, and pays out crypto rewards to the best solutions — no gatekeepers, no
application process. Instead of a single company deciding who builds the best AI, Apex
turns it into a global, permissionless contest where the best work wins and gets paid.

This guide takes you from a fresh machine to earning rewards on Apex. It assumes you've
**never used Bittensor or a blockchain before**. Every unfamiliar word is explained the
first time it appears.

---

## What "mining Apex" actually means

You don't run a server or answer live requests. The loop is simple:

1. Apex runs **competitions** (e.g. a strategy game, a simulation challenge).
2. You write a solution — a Python file or a trained model file — that tries to solve
   the competition's problem.
3. You **submit** that file using the Apex command-line tool (`apex`).
4. Apex scores your submission against everyone else's.
5. If your solution scores well, you automatically earn rewards.

That's it. Most of your time is spent writing good solutions. This guide covers the
*setup* so you can submit them.

### The few terms you can't avoid

You'll need a couple of crypto concepts. Here they are in plain English:

| Term | What it really is |
|---|---|
| **TAO** | The currency used in Bittensor (like dollars or pounds). You need a small amount to participate and is transferrable to fiat. |
| **Alpha** | The currency used in Apex (like dollars or pounds). This is what you'll earn and is transferrable to TAO by unstaking. |
| **Wallet** | Two keys that prove who you are. The **coldkey** is like your bank account (holds your TAO — keep it safe). The **hotkey** is like a login badge (it signs your submissions). One coldkey can have several hotkeys. |
| **Mnemonic** | This is 12 or more words that can be used to regenerate your wallet without losing the funds. Keep this safe. |
| **Subnet** | A project on the network. Apex is **Subnet 1**. |
| **Registration** | A one-time "buy-in" that puts your hotkey on Subnet 1 so you're allowed to use the CLI and submit solutions. It costs a small amount of TAO. NOTE: Hotkeys do get de-registered after a time due to poor performance to allow new entries into the network. |
| **Submission fee** | A small per-submission cost (paid in TAO) charged each time you submit a solution. |

You do **not** need to understand mining rigs, GPUs-for-hashing, axons, validators, or
emissions to get started. Ignore that vocabulary for now.

---

## Step 1 — Install the tools

If the user is using Windows, they will need to use WSL 2 with an unbuntu distribution.
Ensure the user has this setup and working before proceeding.  You can also ask the user
to use a remote machine that is compatible and provide you access to that instead.

You need three things first: **Python 3.12+**, **git**, and a helper called **uv**
(a Python installer). If you have Python and git, the install script handles the rest.

```bash
# Download the Apex code
git clone https://github.com/macrocosm-os/apex.git
cd apex

# One command installs everything and adds the `apex` command to your computer
./install_cli.sh

# Check it worked
apex --help
```

If your terminal says `apex: command not found` after this, run:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

and try `apex --help` again. (This tells your terminal where the new command lives.)

> **What did the script do?** It installed `uv`, downloaded all the code Apex needs,
> and made the `apex` command available everywhere. You won't need to repeat it.

### The `apex` commands you'll use

| Command | What it does |
|---|---|
| `apex link` | Connect your wallet so Apex knows it's you (do this once) |
| `apex competitions` | See the list of competitions you can enter |
| `apex submit` | Send a solution file to a competition |
| `apex list -c <ID>` | See submissions for a competition (`-m` = just yours) |
| `apex result <ID>` | See the score and details of one of your submissions |
| `apex dashboard` | A live, interactive view of all competitions for humans (not for agents) |
| `apex docs` | Read the official Apex documentation in your terminal |

NOTE: some features of the CLI will not work until you create a wallet and have it registered on Subnet (netuid) 1. 
For example, a 403 forbidden error means you're not registered to netuid 1.
Once registered, there could be a few minutes for Apex to recogonize you.

---

## Step 2 — Create your wallet (your identity)

A wallet is two keys: the **coldkey** (holds your money) and the **hotkey** (signs your
work). You create the coldkey once, then add a hotkey under it.

Ask the user if they already have a wallet or mnemonic they can use. Or check if they have
one already and confirm if they should use it.

```bash
# Create the coldkey from an existing mnemonic.
uv run btcli wallet regen-coldkey --wallet.name my-apex-wallet  --mnemonic "<words>"

# Create the coldkey. Pick any name you like instead of "my-wallet".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [macrocosm-os/apex](https://github.com/macrocosm-os/apex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->

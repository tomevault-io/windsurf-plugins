---
trigger: always_on
description: - The `scripts/` directory contains Bash scripts for managing Sui multisig wallets and transactions.
---

# Sui Multisig CLI Scripts & Helpers

## Overview

- The `scripts/` directory contains Bash scripts for managing Sui multisig wallets and transactions.
- These scripts are **for Sui**, not IOTA, and are the authoritative source for Sui multisig CLI operations.
- The web client in `multisig-web-client/` is currently IOTA-specific and **not for Sui**.

---

## Main Workflow Scripts

- **[0_setup_multisig.sh](mdc:scripts/0_setup_multisig.sh)**  
  Interactive script to create a new Sui multisig wallet.  
  - Lets you select addresses, assign weights, set a threshold.
  - Uses `sui keytool multi-sig-address` to generate the multisig address.
  - Saves config as JSON in `multisigs/`.
  - Optionally funds the new address from the faucet.

- **[1_create_tx.sh](mdc:scripts/1_create_tx.sh)**  
  Entry point for creating a new unsigned transaction for a multisig wallet.  
  - Lets you pick the transaction type: `publish`, `upgrade`, `call`, or `transfer`.
  - Delegates to the corresponding script in `scripts/types/`.

- **[2_approve_tx.sh](mdc:scripts/2_approve_tx.sh)**  
  Used by signers to approve/sign a pending transaction.  
  - Lets you pick a transaction, select a signer address, and signs the transaction bytes.
  - Stores signatures in the transaction directory.

- **[3_execute_tx.sh](mdc:scripts/3_execute_tx.sh)**  
  Combines collected signatures and submits the transaction to the Sui network.  
  - Checks if the signature threshold is met.
  - Uses `sui keytool multi-sig-combine-partial-sig` and `sui client execute-signed-tx`.

---

## Transaction Type Scripts

Located in `scripts/types/`:

- **[transfer.sh](mdc:scripts/types/transfer.sh)**  
  Prepares a transfer transaction (object transfer) for a multisig wallet.

- **[call.sh](mdc:scripts/types/call.sh)**  
  Prepares a Move contract call transaction for a multisig wallet.

- **[publish.sh](mdc:scripts/types/publish.sh)**  
  Prepares a package publish transaction for a multisig wallet.

- **[upgrade.sh](mdc:scripts/types/upgrade.sh)**  
  Prepares a package upgrade transaction for a multisig wallet.

---

## Utility Scripts

- **[util/transaction_helpers.sh](mdc:scripts/util/transaction_helpers.sh)**  
  Common Bash functions for address selection, validation, transaction decoding, and more.  
  Used by all main scripts.

- **[util/a_create_keys.sh](mdc:scripts/util/a_create_keys.sh)**  
  Simple script to generate new Sui addresses for all supported key schemes.

---

## Directory Structure

- `multisigs/` — Stores multisig wallet config JSON files.
- `transactions/` — Stores unsigned transaction bytes and collected signatures.

---

## Usage Flow

1. **Create multisig wallet:**  
   Run `scripts/0_setup_multisig.sh` and follow prompts.

2. **Create transaction:**  
   Run `scripts/1_create_tx.sh` and select the type.

3. **Approve transaction:**  
   Each signer runs `scripts/2_approve_tx.sh` to sign.

4. **Execute transaction:**  
   Run `scripts/3_execute_tx.sh` to combine signatures and submit.

---

## Note

- The web client (`multisig-web-client/`) is currently for IOTA and **not compatible with Sui**.  
  All Sui multisig operations should use the CLI scripts above.

# Sui Keytool CLI

The Sui CLI `keytool` command provides several command-level access for the management and generation of addresses, as well as working with private keys, signatures, or zkLogin. For example, a user could export a private key from the Sui Wallet and import it into the local Sui CLI wallet using the `sui keytool import [...]` command.

## Check Sui CLI installation

Before you can use the Sui CLI, you must install it. To check if the CLI exists on your system, open a terminal or console and type the following command:

```bash
$ sui --version
```

If the terminal or console responds with a version number, you already have the Sui CLI installed.

If the command is not found, follow the instructions in Install Sui to get the Sui CLI on your system.

## Commands

Usage: `sui keytool [OPTIONS] <COMMAND>`

Commands:
*   `convert`: Convert private key from legacy formats (e.g. Hex or Base64) to Bech32 encoded 33 byte `flag || private key` begins with `suiprivkey`
*   `decode-or-verify-tx`: Given a Base64 encoded transaction bytes, decode its components. If a signature is provided, verify the signature against the transaction and output the result.
*   `decode-multi-sig`: Given a Base64 encoded MultiSig signature, decode its components. If tx_bytes is passed in, verify the multisig
*   `generate`: Generate a new keypair with key scheme flag {ed25519 | secp256k1 | secp256r1} with optional derivation path, default to m/44'/784'/0'/0'/0' for ed25519 or m/54'/784'/0'/0/0 for secp256k1 or m/74'/784'/0'/0/0 for secp256r1. Word length can be { word12 | word15 | word18 | word21 | word24} default to word12 if not specified
*   `import`: Add a new key to sui.keystore using either the input mnemonic phrase or a private key (from the Wallet), the key scheme flag {ed25519 | secp256k1 | secp256r1} and an optional derivation path, default to m/44'/784'/0'/0'/0' for ed25519 or m/54'/784'/0'/0/0 for secp256k1 or m/74'/784'/0'/0/0 for secp256r1. Supports mnemonic phrase of word length 12, 15, 18, 21, 24

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [multisig-sui/sui-multisig](https://github.com/multisig-sui/sui-multisig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->

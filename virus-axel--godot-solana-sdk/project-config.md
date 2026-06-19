---
trigger: always_on
description: >
---


# Godot Solana SDK

A GDExtension (C++) plugin for Godot 4 that brings full Solana blockchain support to game development.
Write GDScript to send transactions, manage wallets, mint NFTs, and interact with Anchor programs —
on Windows, Linux, macOS, Web, Android, and iOS.

- **Repo:** https://github.com/Virus-Axel/godot-solana-sdk
- **Docs:** https://zenwiki.gitbook.io/solana-godot-sdk-docs
- **Demo:** https://zenrepublic.github.io/GodotSolanaSDKDemos/
- **Asset Library:** https://godotengine.org/asset-library/asset/3232
- **Discord:** https://discord.gg/9aFDCvqPgt

> **Security:** Mainnet use is not yet security-audited. Crypto can be stolen. Use with care.

---

## Installation

**Via Godot Asset Library (recommended)**
1. Search for **"Solana"** in the Godot Asset Library and install.
2. Install must go into the **`addons/`** folder at the project root.
3. Go to **Project → Project Settings → Plugins** and enable **SolanaSDK**.
4. Verify **`SolanaService`** appears under **Project Settings → Autoload**.

**Via GitHub Releases (manual)**
Download binaries from the [Releases page](https://github.com/Virus-Axel/godot-solana-sdk/releases),
place them under `res://bin/`, and reload the project.

---

## Configuration

After enabling the plugin, **`SolanaService`** is automatically added as an autoload singleton.

- **RPC cluster:** Select Mainnet or Devnet in the SolanaService inspector.
  Provide a custom RPC URL (e.g. from [Helius](https://www.helius.dev)) for better performance.
  The URL is also read from `Project Settings → solana_sdk/client/default_url`.
- **Wallet:** The `WalletService` child node handles auth.
  - *Use Generated* — deterministic wallet seeded from your machine ID (testing only).
  - *Custom Wallet* — path to a JSON file with the private key as a 64-byte array.

---

## Core Nodes

| Node | Inherits | Purpose |
|---|---|---|
| `SolanaClient` | `Node` | Low-level RPC calls to the Solana network |
| `Transaction` | `SolanaClient` | Build, sign, and send transactions |
| `WalletAdapter` | `Node` | Browser / mobile wallet integration |
| `Account` | `Node` | Mirror of an on-chain account, auto-syncs |
| `AccountFetcher` | `SolanaClient` | Bulk-fetch a list of Account nodes |
| `SystemProgram` | `Node` | SOL transfers and account creation |
| `TokenProgram` | `Node` | SPL Token instructions |
| `AssociatedTokenAccountProgram` | `Node` | Create Associated Token Accounts |
| `ComputeBudget` | `Node` | Set compute unit limits and priority fees |
| `MplCandyMachine` | `Node` | Candy Machine v3 minting |
| `MplCandyGuard` | `Node` | Candy Guard configuration |
| `MplTokenMetadata` | `Node` | Metaplex token metadata |
| `AnchorProgram` | `Node` | Generic interface for any Anchor program |
| `SolanaUtils` | `Node` | Base58/Base64 encoding, hashing utilities |

## Core Resources

| Resource | Purpose |
|---|---|
| `Pubkey` | 32-byte Solana public key / address |
| `Keypair` | ed25519 keypair for signing |
| `Instruction` | A single transaction instruction |
| `AccountMeta` | Account metadata entry in an instruction |
| `CandyMachineData` | Candy Machine configuration |

---

## Pubkey

```gdscript
# Create from base58 string
var pk: Pubkey = Pubkey.new_from_string("78GVwUb8ojcJVrEVkwCU5tfUKTfJuiazRrysGwgjqsif")

# Create from bytes
var pk: Pubkey = Pubkey.new_from_bytes(some_packed_byte_array)

# Derive a PDA (searches for valid bump, returns null on failure)
var program_key := Pubkey.new_from_string("CndyV3LdqHUfDLmE5naZjVN8rBZz4tqhdefbAnjHG3JR")
var pda: Pubkey = Pubkey.new_pda(["Level1"], program_key)

# Derive a PDA from raw byte seeds
var pda: Pubkey = Pubkey.new_pda_bytes([some_pubkey.to_bytes()], program_key)

# Derive Associated Token Address
var ata: Pubkey = Pubkey.new_associated_token_address(owner, mint, TokenProgram.get_pid())

# Random pubkey (for testing)
var pk: Pubkey = Pubkey.new_random()

# Convert
pk.to_string()           # base58 String
pk.to_bytes()            # PackedByteArray (32 bytes)
```

> `Pubkey.new_pda` finds the canonical valid PDA (searches for bump). `Pubkey.new_program_address`
> does not search — it constructs from exact seeds. Use `new_pda` for normal PDA derivation.

---

## Keypair

```gdscript
# Generate random keypair
var kp: Keypair = Keypair.new_random()

# From a deterministic seed
var seed = PackedByteArray(); seed.resize(32)
var kp: Keypair = Keypair.new_from_seed(seed)

# From Phantom private key (base58 string of 64 bytes)
var kp: Keypair = Keypair.new_from_bytes("3wUbDHMtMVQ...")

# From CLI-style JSON file
var kp: Keypair = Keypair.new_from_file("res://payer.json")

# Save to JSON file
kp.save_to_file("keypair.json")

# Inspect keys
kp.get_public_string()   # base58 String
kp.get_public_bytes()    # PackedByteArray (32 bytes)
kp.get_private_bytes()   # PackedByteArray (64 bytes)
kp.to_pubkey()           # Pubkey resource

# Sign and verify
var sig: PackedByteArray = kp.sign_message("hello".to_ascii_buffer())
var ok: bool = kp.verify_signature(sig, "hello".to_ascii_buffer())
```

---

## SolanaClient (RPC)

> **Important:** `SolanaClient` must be added as a child node (`add_child(client)`) for async
> HTTP requests to work. Remove and free it when done.

**Signals:**
```
http_request_completed(error: Error, response: Dictionary)
socket_response_received
```

```gdscript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Virus-Axel/godot-solana-sdk](https://github.com/Virus-Axel/godot-solana-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

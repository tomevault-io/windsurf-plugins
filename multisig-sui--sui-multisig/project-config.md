---
trigger: always_on
description: Sui supports multi-signature (multisig) transactions, which require multiple keys for authorization rather than a single, one-key signature. In technical terms, Sui supports k out of n multisig transactions, where k is the threshold and n is the total weights of all participating parties. The maximum number of parties is 10. To learn more about the single key signatures that Sui supports, see Signatures.
---

# Sui Multisig Documentation

## Multisig Overview (from Sui Docs)

Sui supports multi-signature (multisig) transactions, which require multiple keys for authorization rather than a single, one-key signature. In technical terms, Sui supports k out of n multisig transactions, where k is the threshold and n is the total weights of all participating parties. The maximum number of parties is 10. To learn more about the single key signatures that Sui supports, see Signatures.

Valid participating keys for multisig are Pure Ed25519, ECDSA Secp256k1, and ECDSA Secp256r1. A (u8) weight is set for each participating keys and the threshold can be set as u16. If the serialized multisig contains enough valid signatures of which the sum of weights passes the threshold, Sui considers the multisig valid and the transaction executes.

### Applications of multisig
Sui allows you to mix and match key schemes in a single multisig account. For example, you can pick a single Ed25519 mnemonic-based key and two ECDSA secp256r1 keys to create a multisig account that always requires the Ed25519 key, but also one of the ECDSA secp256r1 keys to sign. You could use this structure for mobile secure enclave stored keys as two-factor authentication.

**Info:** Currently, iPhone and high-end Android devices support only ECDSA secp256r1 enclave-stored keys.

Compared to threshold signatures, a multisig account is generally more flexible and straightforward to implement and use, without requiring complex multi-party computation (MPC) account setup ceremonies and related software, and any dependency in threshold crypto providers. Additionally, apart from the ability to mix and match key schemes and setting different weights for each key (which is complex in threshold cryptography), multisig accounts are "accountable" and "transparent" by design because both participating parties and observers can see who signed each transaction. On the other hand, threshold signatures provide the benefits of hiding the threshold policy, but also resulting in a single signature payload, making it indistinguishable from a single-key account.

Supported structures in Sui multisig Multisig structures supported in Sui.

### Related links
Multisig Authentication: Guide on how to create a multisig transaction.

## Multisig Authentication (CLI Guide - Excerpt)

The following steps demonstrate how to create a multisig transaction and then submit it against a network using the Sui CLI.

To learn more about how to create multisig addresses and create multisig transactions using the TypeScript SDK, see the SDK documentation for details.

### Prerequisites
This topic assumes you are somewhat familiar with the Sui CLI, specifically the `sui client` and `sui keytool` commands.
You need an existing address on the network you are working on to receive an object.
The topic also assumes that your active environment is Testnet (`sui client active-env`).

### Executing multisig transactions
To demonstrate multisig, this topic guides you through setting up and executing a multisig transaction using the Sui CLI.

#### Create addresses with different schemes
Use the `sui client new-address` command to generate a Sui address and public key for three supported key schemes.
`$ sui client new-address ed25519`
`$ sui client new-address secp256k1`
`$ sui client new-address secp256r1`

Set shell variables for these addresses:
`$ ADDRESS1=<SUI-ADDRESS-ED25519>`
`$ ADDRESS2=<SUI-ADDRESS-SECP256K1>`
`$ ADDRESS3=<SUI-ADDRESS-SECP256R1>`
`$ ACTIVE=<ACTIVE-ADDRESS>`

#### Verify addresses
Use `sui keytool list`. The output includes public key data. Create shell variables for public keys:
`$ PKEY_1=<PUBLIC-KEY-ED25519>`
`$ PKEY_2=<PUBLIC-KEY-SECP256K1>`
`$ PKEY_3=<PUBLIC-KEY-SECP256R1>`

#### Create a multisig address
Use `sui keytool multi-sig-address`. Each address is assigned a weight. The sum of weights of included signatures must be >= threshold.
`$ sui keytool multi-sig-address --pks $PKEY_1 $PKEY_2 $PKEY_3 --weights 1 2 3 --threshold 3`
Response includes `<MULTISIG-ADDRESS>`.

#### Add SUI to the multisig address
Set `$ MULTISIG=<MULTISIG-ADDRESS>`.
Use `sui client faucet --address $MULTISIG`.
Verify with `sui client gas $MULTISIG`.

#### Transfer SUI to your active address
Set `$ COIN=<COIN-OBJECT-ID>`.
Use `sui client transfer --to $ACTIVE --object-id $COIN --serialize-unsigned-transaction`.
Response is `<TX-BYTES-RESULT>`. Set `$ TXBYTES=<TX-BYTES-RESULT>`.

#### Sign the transaction with two public keys
Use `sui keytool sign --address $ADDRESS1 --data $TXBYTES`.
Repeat for `$ADDRESS2`.
Response includes `<SIGNATURE-HASH>`. Set `$ SIG_1=<SIGNATURE-HASH-ED25519>` and `$ SIG_2=<SIGNATURE-HASH-SECP256K1>`.

#### Combine individual signatures into a multisig
Use `sui keytool multi-sig-combine-partial-sig --pks $PKEY_1 $PKEY_2 $PKEY_3 --weights 1 2 3 --threshold 3 --sigs $SIG_1 $SIG_2`.
Response includes `<MULTISIG-SERIALIZED-HASH>`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [multisig-sui/sui-multisig](https://github.com/multisig-sui/sui-multisig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->

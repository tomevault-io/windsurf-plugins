---
trigger: always_on
description: This directory contains unit tests for trait implementations derived by the `#[derive(LightAccount)]` and `#[derive(LightCompressible)]` macros on account data structs.
---

# Account Macros Test Directory

This directory contains unit tests for trait implementations derived by the `#[derive(LightAccount)]` and `#[derive(LightCompressible)]` macros on account data structs.

## Test Coverage Requirement

**Every account struct** with `#[derive(LightAccount)]` or `#[derive(LightCompressible)]` **must have its own dedicated test file** in this directory.

## Directory Structure

```
account_macros/
├── CLAUDE.md                    # This documentation
├── shared.rs                    # Generic test helpers and CompressibleTestFactory trait
├── d1_single_pubkey_test.rs     # Tests for SinglePubkeyRecord
├── d1_multi_pubkey_test.rs      # Tests for MultiPubkeyRecord (TODO)
├── d1_no_pubkey_test.rs         # Tests for NoPubkeyRecord (TODO)
└── ...                          # One test file per account struct
```

## File Naming Convention

Test files follow the pattern: `{dimension}_{struct_descriptor}_test.rs`

- **Dimension prefix** matches the source module (e.g., `d1_` for `d1_field_types/`)
- **Struct descriptor** is a snake_case description of the struct being tested
- **Suffix** is always `_test.rs`

Examples:
| Account Struct | Source Module | Test File |
|----------------|---------------|-----------|
| `SinglePubkeyRecord` | `d1_field_types/single_pubkey.rs` | `d1_single_pubkey_test.rs` |
| `MultiPubkeyRecord` | `d1_field_types/multi_pubkey.rs` | `d1_multi_pubkey_test.rs` |
| `NoPubkeyRecord` | `d1_field_types/no_pubkey.rs` | `d1_no_pubkey_test.rs` |
| `CompressAsAbsentRecord` | `d2_compress_as/absent.rs` | `d2_compress_as_absent_test.rs` |

## Required Test File Structure

Each test file must contain three sections:

### 1. Factory Implementation (Required)

Implement `CompressibleTestFactory` for your struct:

```rust
use super::shared::CompressibleTestFactory;

impl CompressibleTestFactory for YourRecord {
    fn with_compression_info() -> Self {
        Self {
            compression_info: Some(CompressionInfo::default()),
            // ... initialize all other fields with valid test values
        }
    }

    fn without_compression_info() -> Self {
        Self {
            compression_info: None,
            // ... initialize all other fields with valid test values
        }
    }
}
```

### 2. Generic Tests via Macro (Required)

Invoke the macro to generate 17 generic trait tests:

```rust
use crate::generate_trait_tests;

generate_trait_tests!(YourRecord);
```

This generates tests for:
- **LightDiscriminator** (4 tests): 8-byte length, non-zero, method matches constant, slice matches array
- **HasCompressionInfo** (6 tests): reference access, mutation, opt access, set_none, panic on None
- **CompressAs** (2 tests): sets compression_info to None, returns Cow::Owned
- **Size** (2 tests): positive value, deterministic
- **CompressedInitSpace** (1 test): includes discriminator
- **DataHasher** (3 tests): 32-byte output, deterministic, compression_info affects hash

### 3. Struct-Specific Tests (Required)

Tests that cannot be generic because they depend on the struct's specific fields:

#### CompressAs Field Preservation Tests
```rust
#[test]
fn test_compress_as_preserves_other_fields() {
    // Verify each field is preserved after compress_as()
}

#[test]
fn test_compress_as_when_compression_info_already_none() {
    // Verify compress_as() works when compression_info starts as None
}
```

#### DataHasher Field Sensitivity Tests
```rust
#[test]
fn test_hash_differs_for_different_{field_name}() {
    // One test per non-compression_info field
    // Verify changing that field changes the hash
}
```

#### Pack/Unpack Tests (if struct has direct Pubkey fields)

**IMPORTANT**: Only direct `Pubkey` fields are converted to `u8` indices. `Option<Pubkey>` fields are **NOT** converted - they remain as `Option<Pubkey>` in the packed struct.

```rust
#[test]
fn test_packed_struct_has_u8_{pubkey_field}() {
    // Verify PackedX struct has u8 index for each direct Pubkey field
    // Note: Option<Pubkey> fields stay as Option<Pubkey>
}

#[test]
fn test_pack_converts_pubkey_to_index() {
    // Verify Pubkey -> u8 index conversion
}

#[test]
fn test_pack_reuses_same_pubkey_index() {
    // Same Pubkey packed twice gets same index
}

#[test]
fn test_pack_different_pubkeys_get_different_indices() {
    // Different Pubkeys get different indices
}

#[test]
fn test_pack_sets_compression_info_to_none() {
    // Packed struct always has compression_info = None
}

#[test]
fn test_pack_stores_pubkeys_in_packed_accounts() {
    // Verify pubkeys are stored in PackedAccounts
}

#[test]
fn test_pack_index_assignment_order() {
    // Verify sequential index assignment
}
```

## Checklist for Creating a New Test File

When adding tests for a new account struct `MyNewRecord`:

- [ ] Create test file: `{dimension}_{descriptor}_test.rs`
- [ ] Add imports:
  ```rust
  use super::shared::CompressibleTestFactory;
  use crate::generate_trait_tests;
  use csdk_anchor_full_derived_test::{PackedMyNewRecord, MyNewRecord};
  use light_hasher::{DataHasher, Sha256};
  use light_sdk::{
      compressible::{CompressAs, CompressionInfo, Pack},
      instruction::PackedAccounts,
  };

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lightprotocol/light-protocol](https://github.com/Lightprotocol/light-protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

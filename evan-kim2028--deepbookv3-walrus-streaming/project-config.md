---
trigger: always_on
description: Sui Move instructions (.move files):
---

Sui Move instructions (.move files):

- Only put comments to document functions, struct fields, and items that need clarification. DO NOT PUT EXTRANEOUS COMMENTS THROUGHOUT 

- Sui is an object-oriented blockchain. Sui smart contracts are written in the Move language.

- Sui's object ownership model guarantees that the sender of a transaction has permission to use the objects it passes to functions as arguments.

- Sui object ownership model in a nutshell:
  - Single owner objects: owned by a single address - granting it exclusive control over the object.
  - Shared objects: any address can use them in transactions and pass them to functions.
  - Immutable objects: like Shared objects, any address can use them, but they are read-only.

- Abilities are a Move typing feature that control what actions are permissible on a struct:
  - `key`: the struct can be used as a key in storage. If an struct does not have the key ability, it has to be stored under another struct or destroyed before the end of the transaction.
  - `store`: the struct can be stored inside other structs. It also relaxes transfer restrictions.
  - `drop`: the struct can be dropped or discarded. Simply allowing the object to go out of scope will destroy it.
  - `copy`: the struct can be copied.

- Structs can only be created within the module that defines them. A module exposes functions to determine how its structs can be created, read, modified and destroyed.

- Similarly, the `transfer::transfer/share/freeze/receive/party_transfer` functions can only be called within the module that defines the struct being transferred. However, if the struct has the `store` ability, the `transfer::public_transfer/public_share/etc` functions can be called on that object from other modules.

- All numbers are unsigned integers (u8, u16, u32, u64, u128, u256).

- Functions calls are all or nothing (atomic). If there's an error, the transaction is reverted.

- Race conditions are impossible.

- It is allowed to compare a reference to a value using == or !=. The language automatically borrows the value if one operand is a reference and the other is not.

- Integer overflows/underflows are automatically reverted. Any transaction that causes an integer overflow/underflow cannot succeed. E.g. `std::u64::max_value!() + 1` raises an arithmetic error.

- Don't worry about "missing imports", because the compiler includes many std::/sui:: imports by default.

- Don't worry about emitting additional events.

- Prefer macros over constants.

- Put public functions first, then public(package), then private.


## TOOL CALLING INSTRUCTIONS
- `sui move build` to build the package, must be run in a directory with Move.toml in it
- `sui move test` to run tests, must be run in a directory with Move.toml in it
- can pass `--skip-fetch-latest-git-deps` if the dependencies haven't changed after an initial successful build
- when you have completed making changes, run `bunx prettier-move -c *.move --write` on any files that are modified to format them correctly.

# Code Quality Checklist

The rapid evolution of the Move language and its ecosystem has rendered many older practices
outdated. This guide serves as a checklist for developers to review their code and ensure it aligns
with current best practices in Move development. Please read carefully and apply as many
recommendations as possible to your code.

## Code Organization

Some of the issues mentioned in this guide can be fixed by using
[Move Formatter](https://www.npmjs.com/package/@mysten/prettier-plugin-move) either as a CLI tool,
or [as a CI check](https://github.com/marketplace/actions/move-formatter), or
[as a plugin for VSCode (Cursor)](https://marketplace.visualstudio.com/items?itemName=mysten.prettier-move).

## Package Manifest

### Use Right Edition

All of the features in this guide require Move 2024 Edition, and it has to be specified in the
package manifest.

```toml
[package]
name = "my_package"
edition = "2024.beta" # or (just) "2024"
```

### Implicit Framework Dependency

Starting with Sui 1.45 you no longer need to specify framework dependency in the `Move.toml`:

```toml
# old, pre 1.45
[dependencies]
Sui = { ... }

# modern day, Sui, Bridge, MoveStdlib and SuiSystem are imported implicitly!
[dependencies]
```

### Prefix Named Addresses

If your package has a generic name (e.g., `token`) – especially if your project includes multiple
packages – make sure to add a prefix to the named address:

```toml
# bad! not indicative of anything, and can conflict
[addresses]
math = "0x0"

# good! clearly states project, unlikely to conflict
[addresses]
my_protocol_math = "0x0"
```

## Imports, Module and Constants

### Using Module Label

```move
// bad: increases indentation, legacy style
module my_package::my_module {
    public struct A {}
}

// good!
module my_package::my_module;

public struct A {}
```

### No Single `Self` in `use` Statements

```move
// correct, member + self import
use my_package::other::{Self, OtherMember};

// bad! `{Self}` is redundant
use my_package::my_module::{Self};

// good!
use my_package::my_module;
```

### Group `use` Statements with `Self`

```move
// bad!
use my_package::my_module;
use my_package::my_module::OtherMember;

// good!
use my_package::my_module::{Self, OtherMember};
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Evan-Kim2028) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

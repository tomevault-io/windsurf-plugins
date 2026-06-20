---
trigger: always_on
description: Expert guidance for building on the Supra blockchain using Move - contracts, SDK integration, dVRF, Oracles, and Automation.
---


# Supra Move Development Skill

> **You are an expert Supra blockchain Move developer.**
> Supra is a high-performance Layer 1 blockchain with MoveVM + EVM. This skill covers MoveVM only.
> For EVM/Solidity on Supra: https://docs.supra.com/network/evm/

---

## COMPILER RULES - APPLY TO EVERY FILE YOU WRITE

These four rules cause compile errors on every single contract. They are not edge cases. Apply them before writing any code.

---

### RULE 1 - No `///` comments in your contract code. Use `//` only.

When the Supra CLI compiles your Move source, `///` causes a compile error. Use `//` for all comments in files you write.

```move
// WRONG - compile error when compiled with Supra CLI
/// This module does X
/// param: the value

// CORRECT
// This module does X
// param: the value
```

**Note:** You may see `///` in Supra framework source files (e.g. dora-interface oracle files). Those are pre-compiled native modules — they are not passed through the CLI compiler. The ban applies only to `.move` files in your own project's `sources/` directory.

---

### RULE 2 - ASCII only. No Unicode anywhere in source files.

Move only allows ASCII characters (0x20-0x7E). Non-ASCII kills the build **even inside comments**. This includes emoji, symbols, and typographic punctuation.

```
error[E01001]: invalid character
  | // SmartTable used for O(1) lookup — gas stays flat
  |                                    ^ Invalid character '—'

error[E01001]: invalid character
  | // ✅ Caller is whitelisted
  |    ^^ Invalid character '✅'
```

Characters that will always break compilation:

| You might write | Unicode | Use this instead |
|---|---|---|
| `--` (em dash) | U+2014 | `--` |
| `-` (en dash) | U+2013 | `-` |
| `---` (box drawing) | U+2500 | `-` |
| `~=` (almost equal) | U+2248 | `~=` |
| `->` (arrow) | U+2192 | `->` |
| `'` `'` (curly quotes) | U+2018/19 | `'` |
| `"` `"` (curly double quotes) | U+201C/D | `"` |
| `✅` `❌` `⚠️` and any emoji | various | write it out in words |

**Do not use emoji or symbols in comments.** Write `// OK: caller is whitelisted` not `// ✅ caller is whitelisted`. Write `// WARNING:` not `// ⚠️`. No exceptions.

---

### RULE 3 - Type casts: ALWAYS `(expr as T)`. Parens are required.

The Move cast syntax is `(exp as type)`. The parentheses are not optional - they are part of the syntax. Writing `expr as T` without the wrapping parens is always a compile error.

```
error[E01002]: unexpected token
  | let x = (some_expr) as u64;
  |                      ^^ Unexpected 'as'
```

```move
// WRONG - every one of these is a compile error
let a = value as u64;
let b = my_struct.field as u128;
let c = (complex_expr) as u64;
let d = ((a as u128) * b) as u64;

// CORRECT - the entire cast is wrapped in parens
let a = (value as u64);
let b = (my_struct.field as u128);
let c = ((complex_expr) as u64);
let d = (((a as u128) * b) as u64);
```

**Mnemonic:** count your parens. If `as` appears, the IMMEDIATELY enclosing `(` must open right before `expr` and the matching `)` must come after `T`. There must be nothing outside `(expr as T)` except operators or `;`.

---

### RULE 4 - `acquires` list must be exact.

Every function that reads global storage must declare `acquires`. The rules:

1. **Missing = compile error** (`error[E04020]: missing acquires annotation`)
2. **Extra (unused) = compile error** (`error[E02002]: unnecessary or extraneous item`)
3. **Transitive** - if your function calls a helper that acquires `Foo`, YOUR function must also list `Foo`

```move
// Helper acquires ResourceA
fun helper() acquires ResourceA { borrow_global<ResourceA>(@addr); }

// Caller MUST list ResourceA because it calls helper()
public entry fun do_thing() acquires ResourceB, ResourceA {
    let _ = borrow_global<ResourceB>(@addr);  // direct
    helper();                                  // transitive - ResourceA required
}

// WRONG - ResourceA missing (transitive), ResourceC listed but never used
public entry fun do_thing() acquires ResourceB, ResourceC { ... }
```

---

## -- CRITICAL: Move.toml rev = "dev" Warning

> **READ THIS FIRST.** The SupraFramework dependency uses `rev = "dev"`, which tracks the live development branch and can break builds without notice. **Always pin to a specific commit hash for any production deployment.**

```toml
# PRODUCTION - pin to a specific commit
[dependencies.SupraFramework]
git = "https://github.com/Entropy-Foundation/aptos-core.git"
rev = "SPECIFIC_COMMIT_HASH_HERE"
subdir = "aptos-move/framework/supra-framework"

# DEVELOPMENT - live branch, may break
[dependencies.SupraFramework]
git = "https://github.com/Entropy-Foundation/aptos-core.git"
rev = "dev"
subdir = "aptos-move/framework/supra-framework"
```

To find a stable commit hash: browse the commit history at
https://github.com/Entropy-Foundation/aptos-core/commits/dev
or run `git log --oneline -20` inside the Supra CLI container after pulling the latest image.

---

## -- CRITICAL: Always Use supra_framework

```move
// WRONG - will not compile on Supra
use aptos_framework::account;
use aptos_framework::coin;

// CORRECT - always use supra_framework
use supra_framework::account;
use supra_framework::coin;
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Entropy-Foundation/supra-dev-skills](https://github.com/Entropy-Foundation/supra-dev-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

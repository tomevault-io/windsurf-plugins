---
trigger: always_on
description: There are only a few different types of conventions we're concerned about
---

# Conventions

There are only a few different types of conventions we're concerned about

- coding conventions
- repository organization conventions
- generic styx conventions

## Generic Styx Conventions

### Target Program

"Target Program" is the _thing_ being emulated, there are many possible overloaded terms we could choose,
at one point we just called it "firmware," but it is now "target program." This name allows us to accurately
reference the emulated **thing**, while still being able to talk about extra emulated libraries, peripherals,
etc. and not get immediately confused

### Addresses

Speaking of the target program, even if it is a 32 bit program `styx` will always think of any address as 64 bit.
This helps when designing more portables systems and API's, since the only type of address `styx` needs to deal with
is a 64-bit one.

This doesn't mean anything about the target program or depend on any compilation settings, this is just ensuring
that when referencing an address or setting the program counter during emulation that the address is 64 bits wide.

## Programming Conventions

Right now we only stress the Rust conventions, which we attempt to have
a uniform standard by using clippy, rustfmt, and some other uniform rules.

## Rust Code Style Guidelines

The rust styling is enforced by `rustfmt`. The linting by `clippy`, the remainder are a best effort from the
following points:

### Import paths inside the codebase

There are a few different tricks we use in order to keep developer sanity and consistency throughout the codebase. As always with a repository
convention, the important things is keeping it consistent. As long as it is consistent the codebase can all be upgraded together.

The imports are really only different rules for three cases:

- writing styx's core library code (code in `./styx/core`)
- writing styx's non-core library code (code in `./styx` but not in `./styx/core`)
- writing application code that _uses_ styx (code outside `./styx`)

#### Core Library Code

**NOTE**: also see further documentation in `./styx/core/README.md` for other core-specific import invariants that must be followed.

If you are writing code inside of `./styx/core`, you need to import your other `styx_core` counterparts directly by path.

eg.

**Cargo.toml**:

```toml
styx-cpu = { path = "../styx-cpu" }
styx-processor = { path = "../styx-processor" }
```

which would result in possible import like:

```rust
use styx_cpu::Arch;
use styx_processor::Processor;
```

#### Non-core Library Code

**NOTE**: also see further documentation in `./styx/README.md` or `Repository Layout` for import invariants that must be followed.

If you are inside of `./styx` (but not inside `./styx/core`), then you are in one of the styx "non-core"
libraries. The TLDR is that nothing in core can depend on anything outside of core, and that the
dependency cycles should be easily avoided as long as you don't import one processor crate inside
of another processor crate.

The main rules to follow are:

- don't directly import any crate from `core`, instead import `styx-core` and `use` the pieces you need
- if you are adding a new folder under `./styx`, add a new path dependency to the workspace `Cargo.toml`
  - look for the other entries that are similar to: `styx-processors = { path = "./styx/processors" }`
- if you're adding a multi-processor crate, that belongs under the `./styx/machines` sub-directory

An example of importing other libraries when making non-core library modifications:
**Cargo.toml**

```toml
# this is basically always needed
styx-core = { workspace = true }

# other non-core libraries you depend on:
styx-plugins = { workspace = true }
```

So now your import paths could look like:

```rust
use styx_core::prelude::*;
use styx_core::cpu::arch::ArchEndian;
use styx_plugins::tracing::StyxTracePlugin;
```

#### Application Code

Writing code outside of `./styx` is by far the easiest convention to follow, simply add (if still in the `styx` cargo workspace):

**Cargo.toml**

```toml
styx-emulator = { workspace = true }
```

```rust
use styx_emulator::Thing::You::Want::To::Import;

// ~or~
use styx_emulator::prelude::*; // sane default
```

If you are outside the cargo workspace (eg. making your own in-house definitions / library) then you could do the following instead:

**Cargo.toml**

```toml
# get the public `crates.io` version
styx-emulator = { version = "<crates-io version>", features = [ "list", "of", "features" ] }

# ~or~ a specific git version
styx-emulator = { git = "<git repo url>", rev = "<specific git commit, branch, or tag>" }
```

See the [Cargo dependencies](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html) documentation for more.

### Code Formatting and Linting

We employ automatic tools to keep our codebase clean and stylistically consistent:

#### Pre-commit Hooks

Make sure to set up the pre-commit hooks that run rustfmt, cargo check, and cargo clippy. This step is crucial for ensuring that your contributions adhere to our coding standards, `pre-commit` is your friend, not your enemy. Nothing gets merged without passing `pre-commit`.

#### Linting


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [styx-emulator/styx-emulator](https://github.com/styx-emulator/styx-emulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->

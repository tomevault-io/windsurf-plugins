---
trigger: always_on
description: Unified Rust workspace: parses Ghidra `.slaspec` → generates instruction decoder
---

# CLAUDE.md — rsleigh

## What

Unified Rust workspace: parses Ghidra `.slaspec` → generates instruction decoder
+ P-code emitter → decompiles P-code to C pseudocode. Zero C++ deps.
Wired into Spectra as native analysis backend.

Supported: x86-64, x86-32, AArch64, ARM32, MIPS32, RISC-V 64, WASM.
Binary formats: ELF, PE, Mach-O, WASM, raw.

See `docs/architectures.md`, `docs/features.md`, `docs/decompiler-passes.md`.
Testing: `docs/TESTING.md`. SEH: `docs/pe64-seh-pipeline.md`.

## Build

```bash
make test                           # generate + build + test
cargo run -p rsleigh-generate       # parse slaspecs (~30s)
cargo test -p test-harness          # compile + run all tests
```

Rust 2021 stable, make.

## CLI (`rsleigh-cli` lib + `rsleigh` bin)

Install: `cargo install rsleigh` (root crate's bin re-exports `rsleigh_cli::entrypoint`).
Workspace also keeps `cargo install --path rsleigh-cli` working — same binary name `rsleigh`.


```bash
rsleigh <binary>                       # list functions
rsleigh <binary> <func> [func2..]      # decompile (name or 0xAddr)
rsleigh <binary> --all                 # decompile all (two-pass type prop)
rsleigh <binary> --disasm <func>       # disassembly + P-code
rsleigh <binary> --sigs extra.json     # additional signatures
rsleigh <binary> --fid file.fidb       # additional FID database (repeatable)
rsleigh <binary> --no-fid-auto         # disable bundled glibc/musl/libstdc++ DBs
rsleigh <binary> --pcode-json <func>   # raw P-code (debug)
rsleigh <binary> --ssa-json <func>     # post-fold SSA (debug)
rsleigh <binary> --json                # JSON
rsleigh <binary> --search <query>      # find by string/pattern
rsleigh <binary> --search --api <name> # find by API call
rsleigh <binary> --search --const <hex># find by constant
rsleigh <binary> --summary             # one-line per function
rsleigh <binary> --xrefs <func>        # callers + callees
rsleigh <binary> --yara                # generate YARA
rsleigh <binary> --diff <binary2>      # side-by-side diff
rsleigh <binary> --taint <func>        # taint analysis
rsleigh <binary> --vulnscan            # 27 vuln patterns
rsleigh <binary> --ioc [--json]        # IOC extraction (URLs/IPs/paths/registry/mutexes/secrets); see docs/cli-triage.md
rsleigh <binary> --sigcheck [--json]   # Authenticode signer/issuer/timestamp/chain; see docs/cli-triage.md
rsleigh <binary> --resources [--dump DIR] [--json]  # PE resource walk + extraction; see docs/cli-triage.md
rsleigh <binary> --callgraph           # JSON + behavioral tags
rsleigh <binary> --classes [--json]    # C++ hierarchies
rsleigh <binary> --compact             # -24% size
rsleigh <binary> --brief                # calls + cflow only, -35%
rsleigh <binary> --min-complexity N    # skip trivial funcs
rsleigh <binary> --annotate-crypto     # rewrite crypto consts to symbolic names
rsleigh <binary> --vm-dispatch <addr>  # extract VM dispatcher metadata
rsleigh <binary> --vm-classify-handlers <addrs>  # opcode handler classifier
rsleigh <binary> --tag-dispatch <addrs>          # CMP r8/JZ chain extractor
rsleigh <binary> --summarise-handlers <addrs>    # IAT-API + stack-pop signature
rsleigh <binary> --vm-bytecode <bc_va>:<size> --vm-handlers <path.json>  # VM bytecode disasm
rsleigh --raw <arch> <binary>          # raw firmware blob
```

## Layout

```
src/                  parser + SLEIGH codegen (root `rsleigh` crate; also re-exposes CLI bin via src/bin/rsleigh.rs → rsleigh_cli::entrypoint)
pcode-ir/             P-code types + peephole (no_std)
rsleigh-api/          Decoder API + reg name resolution
rsleigh-decompile/    5-pass decompiler (cfg → ssa → fold → structure → print)
rsleigh-fid/          Function ID: body fingerprinting + bundled .fidb
rsleigh-cli/          CLI — split into lib.rs (pub mod cli, wasm; pub use cli::entrypoint) + thin main.rs shim
rsleigh-generate/     slaspec → generated crate source
generated/            output crates (/out/ gitignored)
test-harness/         golden + stress + fuzz + differential
slaspec/              Ghidra .slaspec (Apache 2.0)
scripts/              Ghidra/Qt sig extraction, FID DB build
docs/                 detail docs (architectures, features, passes, SEH, testing)
```

## Pipeline

```
.slaspec → parser → codegen → generated crates → compile
bytes + addr → Decoder::decode() → Instruction { disasm, ops: Vec<PcodeOp> }
              → decompile_with_binary() → CFG → SSA → fold → structure → C
```

## Load-bearing gotchas

### Codegen (`src/codegen/builder/disassembler/constructor/execution.rs`)

- Subtable cache: `lift()` once per subtable, results cached
- Unique offset scheme: parent uses `(num_fields*2+2)*0x10000` to avoid subtable-export collision
- `dynamic_value_expr()` resolves aliased token fields by bit position (r32/r64 share bits 0-2)
- Signed displacements: cast signed token fields (simm8, simm16) to signed type before i128 widen
- Const-space refs: `export *[const]:4 simm8` → `Varnode::constant()` (no Load)
- MixOperations: mixed AND/OR pattern blocks default to AND (VFP/NEON)
- Optional table lift: OR-pattern subtables lifted via `.as_ref().unwrap()`

### Decompiler

See `docs/decompiler-passes.md` for full pass list. Hotspots:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShaneBreazeale/rsleigh](https://github.com/ShaneBreazeale/rsleigh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->

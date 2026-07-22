---
trigger: always_on
description: This is the canonical onboarding and development guide for Patchestry.
---

# Patchestry Developer Guide (Canonical)

This is the canonical onboarding and development guide for Patchestry.

- `AGENTS.md` is the source of truth for Codex/Cursor and human contributors.
- `CLAUDE.md` exists only as a compatibility entrypoint for Claude Code and points here.

## Purpose

Patchestry is an MLIR/CIR-based binary patching framework for patching deployed firmware
without original source code.

Important terminology:

- In this document, "patch" means a firmware patch authored against a target firmware image.
- It does not mean a patch to the Patchestry repository itself.

## Architecture and Modules

### End-to-end data flow

```text
Firmware Binary
  -> Ghidra headless scripts (`scripts/ghidra/*`)
  -> P-Code JSON (`*.json`, Ghidra export schema)
  -> `patchir-decomp`
  -> CIR (`*.cir`)
  -> `patchir-transform` + YAML spec (`*.yaml`)
  -> Patched CIR (`*.cir`)
  -> `patchir-cir2llvm`
  -> LLVM IR (`*.ll`) or bitcode (`*.bc`)
  -> downstream binary rewriting / verification tools
```

### Project-owned module inventory

This inventory is intended to be exhaustive for patchestry-owned code. It does
not attempt to document LLVM/MLIR internals or vendored dependency internals.

#### Core libraries and dialects

| Module | Build target | Main paths | Primary role |
|---|---|---|---|
| Ghidra model and translation | `patchestry_ghidra` | `include/patchestry/Ghidra/`, `lib/patchestry/Ghidra/` | deserialize Ghidra JSON and register P-Code translation |
| AST lifting | `patchestry_ast` | `include/patchestry/AST/`, `lib/patchestry/AST/` | lift Ghidra model into Clang AST/CIR-ready structures |
| Codegen | `patchestry_codegen` | `include/patchestry/Codegen/`, `lib/patchestry/Codegen/` | serialize and lower internal representations during tool pipelines |
| YAML parsing | `patchestry_yaml` | `include/patchestry/YAML/`, `lib/patchestry/YAML/` | parse patch and contract YAML configuration |
| Patch passes | `patchestry_passes` | `include/patchestry/Passes/`, `lib/patchestry/Passes/` | apply patch and contract transformations to CIR |
| Contracts dialect | `MLIRContracts` | `include/patchestry/Dialect/Contracts/`, `lib/patchestry/Dialect/Contracts/` | represent contract attributes and verification metadata |
| Pcode dialect | `MLIRPcode` | `include/patchestry/Dialect/Pcode/`, `lib/patchestry/Dialect/Pcode/` | represent and deserialize P-Code as an MLIR dialect |
| Intrinsics library | `patchestry_intrinsics` | `include/patchestry/intrinsics/`, `lib/patchestry/intrinsics/` | provide patch helper/runtime functions for patch C code |
| Utility headers | no standalone target | `include/patchestry/Util/` | shared logging, diagnostics, common options, helper types |

#### Tools

| Tool | Build target | Main paths | Purpose |
|---|---|---|---|
| `patchir-decomp` | `patchir-decomp` | `tools/patchir-decomp/` | decompile Ghidra JSON into CIR/LLVM/asm/object outputs |
| `patchir-transform` | `patchir-transform` | `tools/patchir-transform/` | apply YAML-defined patches and contracts to CIR |
| `patchir-cir2llvm` | `patchir-cir2llvm` | `tools/patchir-cir2llvm/` | lower CIR to LLVM IR or bitcode |
| `patchir-yaml-parser` | `patchir-yaml-parser` | `tools/patchir-yaml-parser/` | validate and inspect YAML configuration |
| `pcode-translate` | `pcode-translate` | `tools/pcode-translate/` | standalone P-Code translation driver built on patchestry Ghidra translation |

#### Workflow scripts

| Script area | Main paths | Purpose |
|---|---|---|
| Ghidra automation | `scripts/ghidra/` | build headless container, run decompilation, serialize functions/P-Code |
| JSON rendering helper | `scripts/render_json.py` | utility script for rendering/inspecting JSON artifacts |

### Module interface map

These interfaces are the contracts contributors should keep stable while iterating on internals.

| Module | Main paths | Stable interface files | Input format | Output format | Module test command |
|---|---|---|---|---|---|
| Ghidra model | `include/patchestry/Ghidra/`, `lib/patchestry/Ghidra/` | `include/patchestry/Ghidra/JsonDeserialize.hpp`, `include/patchestry/Ghidra/Pcode.hpp`, `include/patchestry/Ghidra/PcodeTranslation.hpp` | Ghidra export JSON | in-memory Program/Function/Block/Op model | `lit ./builds/default/test/ghidra -D BUILD_TYPE=Debug -v` |
| AST lifting | `include/patchestry/AST/`, `lib/patchestry/AST/` | `include/patchestry/AST/ASTConsumer.hpp`, `include/patchestry/AST/FunctionBuilder.hpp`, `include/patchestry/AST/OperationBuilder.hpp`, `include/patchestry/AST/TypeBuilder.hpp` | Ghidra model objects | Clang AST and CIR-ready structures | `lit ./builds/default/test/patchir-decomp -D BUILD_TYPE=Debug -v` |
| Codegen | `include/patchestry/Codegen/`, `lib/patchestry/Codegen/` | `include/patchestry/Codegen/Codegen.hpp`, `include/patchestry/Codegen/PassManager.hpp`, `include/patchestry/Codegen/Serializer.hpp` | AST/CIR owned by patchestry tools | serialized/lowered outputs consumed by tool frontends | `lit ./builds/default/test/patchir-decomp -D BUILD_TYPE=Debug -v` |
| Decompiler tool | `tools/patchir-decomp/` | `tools/patchir-decomp/main.cpp` | P-Code JSON | CIR / LLVM IR / asm / object output selected by flags | `lit ./builds/default/test/patchir-decomp -D BUILD_TYPE=Debug -v` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lifting-bits/patchestry](https://github.com/lifting-bits/patchestry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->

---
trigger: always_on
description: - **Never change directory with `cd` when executing commands, stay in the project root (where this file lives)**
---

# Etch Language Implementation

## Directives
- **Never change directory with `cd` when executing commands, stay in the project root (where this file lives)**
- **Never commit or stash code using git, use it only to inspect previous commits or changed files only**
- **Never ever think about giving up, like "we should do Y but because it's too much, for now we do X" and you end up shortcutting to tech debt and fake implementations**
- **When you create new tests in examples (both passing and failing) add the corresponding validation files .pass and .fail**
- **Everytime you are modifying the bytecode or AST structure, bump their corresponding version numbers but keep supporting only the latest version in code**
- **Never disable features if you cannot make them work, instead restart from a smaller problem and iterate in steps until you can understand the issue deeply**
- **Make all examples grouped by the first word, for example if you are testing a specific feature (like `comptime` or `inference`), use it as first part of the example name like `comptime_inject_test.etch`**
- **When hunting bugs or making changes, add extensive verbose logging protected by the verbose flag, this will help you detect issues**
- **Never swallow nim exceptions, print the exception for debugging purposes**
- **Keep updated the documentation in @docs/ (and the developers docs in @docs/internal/) as we keep iterating changes**
- **Remember `result` is a valid keyword in nim, so don't create variables with that name if it's not the implicit one**
- **Try to not add too many comments in methods, try to keep it just for method signature, not several inside bodies at every instruction**
- **Never try to write files into /tmp**
- **Never execute binaries from ~/.cache/nim**
- **Don't waste time with `nimpretty`**

## Testing and validation of correctness

For bugfixing and validation, you have to:
- **Compile and run a single etch file in verbose mode with**:
  - `just go-quiet examples/simple_test.etch` and `just go-quiet examples/simple_test.etch --release` to just test output
  - `just go-compiler examples/simple_test.etch` and `just go-compiler examples/simple_test.etch --release` to print the compiler verbose output
  - `just go-verbose examples/simple_test.etch` and `just go-verbose examples/simple_test.etch --release` to print the full verbose output (compiler + vm)
- **Validate a test case by compiling and running a single example** `just test examples/simple_test.etch`:
    * If the test is passing, you need to provide a `examples/simple_test.pass` file
    * Otherwise a `examples/simple_test.fail` one when it fails.
- **Primary validation testing workflow**: test compiling and running examples + debugger tests with `just tests` and `just tests-c` for C backend tests.
- To test **only** the debugger integration, use `just tests-debugger`.
- Compile and reinstall the VSCode extension with `just vscode`

## Improvement plan

1. Expand immediate & fused forms
In regvm_compiler.nim, emit op*I, op*Int/Float, and comparison+jump opcodes whenever prover/lifetime metadata proves operand types. Mirror Lua’s docondjump by folding the branch into the test instruction (opCmpJmp) so the interpreter executes one dispatch instead of two.

2. Table/array fast paths
Implement helpers akin to luaV_fastget/luaV_fastset in exec_instructions/objects.nim that assume constant short-string keys (opGetField, opSetField) and only fall back to the slow path/metamethod code when a tagged miss occurs. Likewise add specialized opGetI, opSetI variants that bypass hash lookups entirely.

3. Optimizer + prover integration
Wire expression_analysis.nim results into optimizeConstantFolding and a fixed optimizeDeadCode pass so redundant refcount instructions and type guards disappear before emission. Follow AGENTS.md Phase 1 by stripping bounds/division checks the prover proves safe and by resurrecting peephole fusion (Phase 3) to shrink instruction count.

4. Reference-count pressure relief
Extend regvm_lifetime data so optimizeRefCounting can drop adjacent opIncRef/opDecRef pairs and hoist opCheckCycles out of inner loops. Long term, treat heap objects like Lua tables: embed mutability bits and emit barriers only when a write crosses generations.


Etch Language/Compiler/Prover/Interpreter Improvement Plan
Based on the comprehensive VM architecture analysis document, here's a prioritized plan to improve Etch:

Phase 1: Bytecode Optimization Infrastructure (HIGH PRIORITY)

1.1 Integrate Prover Information into Optimizer

Files: src/etch/bytecode/compiler.nim, src/etch/bytecode/prover/expression_analysis.nim
- Use range analysis results from the prover to eliminate runtime checks
- Add dead code elimination based on evaluateCondition() results
- Remove bounds checks when prover proves safety
- Remove division-by-zero checks when prover proves divisor is non-zero
- Impact: Major performance boost, cleaner bytecode

Phase 2: Instruction Set Improvements (MEDIUM PRIORITY)

2.1 Add Jump Target Table

File: src/etch/core/vm.nim
- Create explicit jump target table for each function
- Validate jumps only go to valid targets
- Enable "switch out data tables" optimization mentioned in the document

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ByteSmelt/etch](https://github.com/ByteSmelt/etch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->

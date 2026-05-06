---
trigger: always_on
description: This project is a custom high-performance 64-bit LoongArch emulator with a flat memory arena as memory instead of virtual paging, designed otherwise in the same way as libriscv. Static 64-bit LoongArch ELFs are loaded, executed and after execution one can vmcall functions in the guest.
---

# Project information

This project is a custom high-performance 64-bit LoongArch emulator with a flat memory arena as memory instead of virtual paging, designed otherwise in the same way as libriscv. Static 64-bit LoongArch ELFs are loaded, executed and after execution one can vmcall functions in the guest.

**Note: Only 64-bit LoongArch (LA64) is supported. 32-bit support (LA32) has been removed.**

## Project structure

It's a CMake project with this structure:
- /lib/libloong has the C++ library implementation
    - Instruction templates are in /lib/libloong/la_instr_impl.hpp and /lib/libloong/la_instr_atomic.hpp
	- Instruction printers (for debugging) is in /lib/libloong/la_instr_printers.hpp
	- Instruction decoding is in /lib/libloong/la64.cpp
- /tests/unit has the unit tests (and a run_unit_tests.sh)
    - Unit tests build temporary C and C++ programs and execute them in various ways in the emulator
- /emulator is a C++ CLI with build.sh producing .build/laemu
    - laemu runs forever (or until completion) unless -f <instruction count> is passed
	- it uses a different, faster dispatch than the debugger
- /tests/debug_test.cpp is a full-fledged CLI debugger

## How to debug

`./tests/debug_test` is a comprehensive debugger built in the build folder. It prints each instruction as it executes. Passing -r will also show register state *after* each instruction. Passing -o will compare against ground-truth objdump in case an instruction is incorrectly decoded.

### Examples

Execute `return_42_bare` until completion, compare instructions (left) against objdump (right):
```sh
build$ ./tests/debug_test -o tests/loongarch_bins/return_42_bare
```

Execute `return_42_bare` until completion, show registers after each instruction, compare instructions (left) against objdump (right):
```sh
build$ ./tests/debug_test -r -o tests/loongarch_bins/return_42_bare
```

Execute `cxx_test` until completion without debug lines, then start executing the guest-side function `test_exception` with debugging. This saves a bunch of time and prevents logspam. Only debug lines from test_exception will be shown.
```sh
build$ ./tests/debug_test -o tests/loongarch_bins/cxx_test --call test_exception
```

WARNING: Do _NOT_ use debug_test for benchmarks such as stream.elf and coremark.elf. They will never complete as they run through billions of instructions. Use the CLI.

## Adding a new bytecode

A single bytecode means modifying 5 different places:
1. Adding the new bytecode in threaded_bytecodes.hpp
2. Select which instruction the bytecode replaces in decoder_cache.cpp
3. Implementing an optimized instruction layout in threaded_rewriter.cpp
4. Implementing the actual bytecode handler in bytecode_impl.cpp
5. Adding an entry connecting the bytecode to the handler in threaded_bytecode_array.hpp as well as tailcall_bytecode_array.hpp

Only once all 5 steps are complete can the CLI be tested. Bytecodes should be implemented according to popularity as LoongArch has very many instructions. Any bytecode where register zero could be written to can be rewritten in the optimizer to either NOP or INVALID (bytecode==0) when rd == 0, so that a pointless check for rd != 0 is avoided in hot-path.

Many instructions have slow-path instructions already decoded in la64.cpp and implemented in la_instr_impl.hpp, which can be used as a starting point. We generally don't implement any atomic instructions as bytecodes, as they just occupy instruction space. For LASX instructions, only the most popular ones should have dedicated bytecodes. Instructions without dedicated bytecodes will still function in fast-path, but will execute through the FUNCTION bytecode which adds function call overhead.

Unpopular instructions might be considered for removal from bytecode dispatch because performance is critical.

## Adding a new LSX or LASX instruction

Example:
711e0000 vpickev.b              VdVjVk
711e8000 vpickev.h              VdVjVk
711f0000 vpickev.w              VdVjVk
711f8000 vpickev.d              VdVjVk
Above are the base opcodes for vector 4 instructions in the same family. If they operate on integers, they typically have all 4 variants. Almost all vector instructions have the same format: instr.r3.rd, instr.r3.rj, instr.r3.rk as shown in la_instr.hpp

  2007ac:       711e8800        vpickev.h       $vr0, $vr0, $vr2
  2007b4:       711e8c21        vpickev.h       $vr1, $vr1, $vr3
  2007b8:       711e0020        vpickev.b       $vr0, $vr1, $vr0

Above is an example from objdump showing 3 occurences of these instructions for verification.

LSX instructions go in la_instr_impl.hpp since they are always enabled, but LASX go in la_instr_lasx.hpp. We don't currently care about LASX register zero-extension.

Especially LSX or LASX instructions with subtypes B, H, W and D must share an instruction printer in order to reduce code footprint. Example:
```cpp
	INSTRUCTION_P(VSEQI_B, VSEQI);
	INSTRUCTION_P(VSEQI_H, VSEQI);
	INSTRUCTION_P(VSEQI_W, VSEQI);
	INSTRUCTION_P(VSEQI_D, VSEQI);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libriscv/libloong](https://github.com/libriscv/libloong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->

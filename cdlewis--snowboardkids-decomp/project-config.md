---
trigger: always_on
description: This is a matching decompilation project for Snowboard Kids (N64). The goal is to create C code that, when compiled, produces the exact same ROM as the original game.
---

# CLAUDE.md

## Repository Overview

This is a matching decompilation project for Snowboard Kids (N64). The goal is to create C code that, when compiled, produces the exact same ROM as the original game.

## Project Structure

- `src` decompiled or partially decompiled C code.
- `include` shared headers and assembly macros.
- `asm/nonmatchings` unmatched assembly functions extracted from the ROM.
- `asm/data` extracted data assembly.
- `assets` binary asset blobs extracted from the ROM.
- `symbol_addrs.txt` project symbol names and addresses.
- `snowboardkids.yaml` Splat configuration for ROM extraction and segment layout.

## Tools

- `./tools/build-and-verify.sh` rebuilds extracted assets/asm, builds the ROM, and verifies the SHA1 checksum.
- `python3 tools/asm-differ/diff.py --no-pager <function name>` compares compiled assembly against the target for a specific function. Note that you *must* rebuild the project manually before digging or you will potentially operate on stale data.
- `./tools/claude --bootstrap-only <function name>` creates a per-function matching workspace under `nonmatchings/`.
- `./build.sh <file>.c` inside a generated matching workspace compiles one attempt and compares it to the target function.
- `python3 tools/data-differ/data_diff.py <symbol>` or `./tools/diff-data <symbol>` compares binary data between the target ROM and compiled output for a specific data symbol.
- `python3 tools/data-differ/data_diff.py --find-first-mismatch` scans data symbols in ROM order and shows the first mismatch. Use this after checksum failures when a data variable may be responsible.

## Tasks

### Decompile Assembly to C Code

You may be given a function and asked to decompile it to C code.

#### Step 1

First we need to spin up a decomp environment for the function, run:

```
./tools/claude --bootstrap-only <function name>
```

Move to the directory created by the script. This will be `nonmatchings/<function name>-<number (optional)>`.

Use the tools in this directory to match the function. You may need to make several attempts. Each attempt should be in a new file (`base_1.c`, `base_2.c`, ... `base_n.c`, etc).

#### Step 2 (successful match, integrate changes into project)

If you are able to match the function, update the C code to use it. The C code will be importing an assembly file, something along the lines of `INCLUDE_ASM/asm/nonmatchings/<function name>`. Replace this with the actual C code.

- Update the rest of the project to fix any build issues.
- If the function is defined in a header file (located in include/), this will also need to be updated. These other usages may teach you about the correct type of your function arguments or return types. DO NOT JUST MAKE EVERYTHING void\*!.
- Make sure to search for any existing function / struct declarations in the project (under src/ and include/). We do not want duplicate or redundant declarations.

Verify that the project still builds successfully by running `./tools/build-and-verify.sh`. If this check fails, the decompilation is NOT complete, even if individual functions appear to match.

- If the checksum fails after your changes, use `python3 tools/asm-differ/diff.py --no-pager <function>` to check ALL functions in the modified file(s). Look for functions that access the same structs you modified. Fix any mismatches before declaring success.

#### Step 3: Commit your successful match or improvement on existing partial match

If you are able to get a perfect matching decompilation, commit the changes with an appropriate message describing what you matched and how.

If you are unable to get a perfect match but have improved upon the previous best match (or there is no such previous match), record your best match next to the included ASM:

```
// your_function best match: XX%

#pragma GLOBAL_ASM("asm/nonmatchings/segment/your_function")

#ifdef NON_MATCHING
// your non-matching function
#endif
```

Respect any pre-commit hooks that prevent you from committing your change. A failed hook indicates that you have not correctly updated the C code.

You are done. Do not attempt to find the next closest match.

#### When to stop

**These rules are mandatory. Do not override them.**

If the build script tells you to stop, you MUST stop immediately. Do not make another attempt. Report:
- Best score and which file achieved it
- Summary of approaches tried
- Analysis of remaining differences
- Whether the remaining issues seem solvable or may need a different strategy

## Validation Checklist

Before declaring any changes to C code complete (including decompiling functions), verify:

- [ ] No pointer arithmetic with manual offset calculations
- [ ] All struct field accesses use `->` or `.` operators
- [ ] No `void*` parameters that should be typed structs
- [ ] Struct sizes match the assembly access patterns
- [ ] `./tools/build-and-verify.sh` succeeds


### Capture Learnings


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cdlewis/snowboardkids-decomp](https://github.com/cdlewis/snowboardkids-decomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->

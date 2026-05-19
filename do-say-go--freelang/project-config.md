---
trigger: always_on
description: This guide documents processes and patterns for AI agents/collaborators working on the Freelang compiler project.
---

# Agent Guide for Freelang Development

This guide documents processes and patterns for AI agents/collaborators working on the Freelang compiler project.

## Table of Contents

1. [Testing & Debugging](#testing--debugging)
2. [Windows PE Backend Development](#windows-pe-backend-development)
3. [Common Issues & Solutions](#common-issues--solutions)
4. [Project Structure](#project-structure)

## Testing & Debugging

### Running Tests

The project has multiple ways to run tests:

1. **Full test suite** (expensive, ~2-5 minutes):
   ```powershell
   pwsh ./tests/run-all.ps1 > test-output.txt 2>&1
   ```
   Always save output to a file to enable analysis without re-running.

2. **Subset of tests**:
   ```powershell
   pwsh ./tests/run-all.ps1 --hide-passes
   ```

3. **Single test file**:
   ```powershell
   pwsh flx.ps1 tests/test-name.flx
   ```

### Analyzing Test Failures

After running the full suite with output saved:

1. **Check summary** (last 10-20 lines):
   ```powershell
   Get-Content test-output.txt -Tail 20
   ```

2. **Count specific error patterns**:
   ```powershell
   # Count access violations (crashes)
   Get-Content test-output.txt | Select-String "Actual:\s+-1073741819" | Measure-Object

   # Find linker errors
   Get-Content test-output.txt | Select-String "unresolved external symbol"
   ```

3. **Common exit codes on Windows**:
   - `-1073741819` (0xC0000005): Access violation / segfault
   - `1120`: Linker error (unresolved symbols)
   - `0`: Success

### Debugging Individual Tests

Start with the simplest failing test to understand the root cause:

```powershell
# Test a basic case
pwsh flx.ps1 tests/assert_pass.flx

# Test specific features
pwsh flx.ps1 tests/arrays-basic.flx
pwsh flx.ps1 tests/ir-string-concat.flx
```

Look for linker errors in the output - these directly indicate missing implementations.

## Windows PE Backend Development

### Architecture Overview

The Windows PE backend is in `backend/codegen-x86_64-pe.js` and generates x86-64 assembly for Windows using:
- **Calling Convention**: Microsoft x64 ABI (rcx, rdx, r8, r9 for args)
- **Linker**: lld-link (LLVM linker)
- **Target**: clang -target x86_64-pc-windows-msvc
- **Dependencies**: Only Kernel32.dll (no libc)

### Key Differences from macOS Backend

| Aspect | macOS (System V ABI) | Windows (MS x64 ABI) |
|--------|---------------------|---------------------|
| Args 1-2 | rdi, rsi | rcx, rdx |
| Args 3-4 | rdx, rcx | r8, r9 |
| Stack args | 8(%rsp)+ | 32(%rsp)+ (shadow space) |
| Shadow space | None | 32 bytes required |

### Adding Missing Intrinsics

When you see linker errors like:
```
unresolved external symbol _rt_string_concat
unresolved external symbol _rt_array_concat
unresolved external symbol _rt_object_merge
unresolved external symbol _rt_value_shape_plus
```

**Step 1: Check if the `_freelang_*` version exists**

The codebase uses both naming conventions:
- `_rt_*`: Called from IR's `CallRuntime` instructions
- `_freelang_*`: Internal implementation names

Search for existing implementations:
```javascript
// In backend/codegen-x86_64-pe.js
function emitStringConcat(asm) {
  asm.label('_freelang_string_concat');
  // Implementation...
}
```

**Step 2: Add runtime aliases**

Create label aliases that jump to the implementation:
```javascript
function emitRuntimeAliases(asm) {
  asm.label('_rt_string_concat');
  asm.instr('jmp', '_freelang_string_concat');

  asm.label('_rt_array_concat');
  asm.instr('jmp', '_freelang_array_concat');

  // ... etc
}
```

**Step 3: Implement missing `_freelang_*` functions**

If the `_freelang_*` implementation is missing or stubbed:

1. Check the macOS backend (`backend/codegen-x86_64-macho.js`) for reference
2. Adapt the calling convention from System V to MS x64
3. Key changes needed:
   - Replace `%rdi, %rsi` with `%rcx, %rdx` for first 2 args
   - Add shadow space: `subq $40, %rsp` at function start
   - Use `callq _freelang_memcpy` instead of `rep movsq` if needed

Example adaptation (array_concat):
```javascript
// macOS version uses:
asm.instr('movq', '%rdi', '%r8');    // arr1 from rdi
asm.instr('movq', '%rsi', '%r9');    // arr2 from rsi

// Windows version uses:
asm.instr('movq', '%rcx', '%r8');    // arr1 from rcx
asm.instr('movq', '%rdx', '%r9');    // arr2 from rdx
```

**Step 4: Update buildFromIR to emit functions**

In the `buildFromIR` function, ensure the new intrinsics are emitted:

```javascript
// Around line 266-286
const needsValueOps = features.arrays || features.strings || features.objects;
if (needsValueOps) {
  emitArrayConcat(asm);
  emitStringConcat(asm);
  emitObjectMerge(asm);
  emitRuntimeAliases(asm);
  emitValueShapePlus(asm);
}
```

**Important**: `_rt_value_shape_plus` calls all three concat/merge functions, so they must all be emitted together even if only one feature is used.

### Implementing _rt_value_shape_plus

This intrinsic implements dynamic `+` operator dispatching:

```javascript
function emitValueShapePlus(asm) {
  asm.label('_rt_value_shape_plus');

  // 1. Check if both operands are arrays -> call _freelang_array_concat
  // 2. Check if both operands are strings -> call _freelang_string_concat

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DO-SAY-GO/freelang](https://github.com/DO-SAY-GO/freelang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->

---
trigger: always_on
description: |
---


# CodeRunner Question Generator (Minimal Pipeline)

Generate Moodle CodeRunner questions. Two modes depending on your environment.

## Mode Detection

**Before doing anything else, determine your mode:**

Run this command:
```bash
echo "jobe-check"
```

- **If the command succeeds** (you can execute bash): use **FULL PIPELINE MODE** (Sections 3-9). Jobe computes all expected output via curl. This is the preferred mode.
- **If the command fails** (no bash access, e.g. Claude.ai or ChatGPT): use **FALLBACK MODE** (see box below). You must mentally trace expected output yourself.

> **FALLBACK MODE (no bash access)**
>
> When you cannot run bash commands (Claude.ai, ChatGPT, API without tools):
> 1. Generate the same JSON structure as Section 6, but you MUST include `"expected"` in each test case by carefully tracing the solution through the test code.
> 2. Trace character-by-character. Pay special attention to: trailing spaces, newlines, floating-point precision, array formatting.
> 3. Apply all per-language rules from Section 2 and auto-fix rules from Section 8 manually.
> 4. Wrap in XML using the template from Section 7.
> 5. **Add this warning to the user:** "These questions were generated WITHOUT Jobe server validation. Before using them, import into Moodle and ensure `Validate on save` is enabled (it is by default). Moodle will run the solution against all test cases on import and flag any mismatches."
> 6. All other sections (question design, type rules, XML template, checklist) still apply.

**Full Pipeline mode is strongly preferred.** It eliminates the #1 source of errors (hallucinated expected output, which causes 35% of all failures).

---

## 1. Question Type Reference

Choose the type FIRST -- it determines everything else.

| Type | Student Writes | Test Mechanism | Stdin? | Reliability |
|------|---------------|----------------|--------|-------------|
| `java_class` | A class | Test code calls methods | No | HIGH |
| `java_method` | Static method(s) ONLY | Test code calls methods | No | HIGH |
| `java_program` | Full program with `main` | Empty test, stdin input | Yes | LOW -- EOF issues |
| `python3` | Function(s) or class | Test code calls with `print()` | No | HIGH |
| `python3_w_input` | Full program | Empty test, stdin input | Yes | MEDIUM -- EOF issues |
| `c_function` | Function(s) + headers | Test code has `main()` | No | HIGH |
| `c_program` | Full program with `main` | Empty test, stdin input | Yes | MEDIUM |
| `cpp_function` | Function(s) + headers | Test code has `main()` | No | MEDIUM -- Werror |
| `cpp_program` | Full program with `main` | Empty test, stdin input | Yes | MEDIUM -- Werror |
| `nodejs` | Function(s) | Test code uses `console.log()` | No | HIGH |

**Default choice**: Prefer `java_class` > `java_method` > `java_program`. Prefer function types over program types -- they avoid stdin/EOF problems entirely.

---

## 2. Per-Language Rules

These rules are derived from 1000+ validated questions. Every rule exists because questions failed without it.

### Java

| Rule | Applies To | What To Do |
|------|-----------|------------|
| Scanner EOF guard | `java_program` | ALWAYS call `hasNextLine()`/`hasNextInt()` before EVERY `Scanner` read. #1 Java failure cause. |
| Class name | `java_program` | Class MUST be named `Answer` |
| Method-only solution | `java_method` | Solution MUST be ONLY the method(s) -- NO class wrapper, NO main(), NO import statements. The buildSource step wraps it in `public class Answer { <methods> main() { testcode } }`. Including `public class` or `main()` causes compilation errors. |
| Class qualifier in tests | `java_class` | Test code runs in a separate `__Tester__` class. Call static methods as `ClassName.method()`. Create objects as `ClassName obj = new ClassName(); obj.method()`. |

### Python

| Rule | Applies To | What To Do |
|------|-----------|------------|
| EOF guard | `python3_w_input` | Use `import sys; data = sys.stdin.read().strip()` with `if data:` guard. Never bare `input()`. |
| Function-only solution | `python3` | Solution is function/class definitions only. Test code calls them with `print()`. |

### C

| Rule | Applies To | What To Do |
|------|-----------|------------|
| `-Werror` | ALL C types | Jobe compiles with `-Werror`. ALL warnings are fatal. No unused variables, no implicit declarations. |
| `#include` in answer | `c_function` | ALL headers (`<stdio.h>`, `<ctype.h>`, `<string.h>`, `<math.h>`, `<stdlib.h>`) MUST be in the solution, not just test code. |
| `scanf` return check | `c_program` | Always: `if (scanf("%d", &n) == 1) { ... }`. Unchecked scanf on empty stdin = uninitialized variable. |
| Test code wrapper | `c_function` | Test code MUST include `#include <stdio.h>` and `int main(void) { ... return 0; }` |
| Array printing | ALL C types | Use `if (i) printf(" "); printf("%d", arr[i]);` -- NOT `printf("%d ", arr[i])` (trailing space fails). |
| Function-only solution | `c_function` | Solution is function(s) with #include headers only. NO main(). Test code provides main(). |

### C++

| Rule | Applies To | What To Do |
|------|-----------|------------|
| `-Werror` | ALL C++ types | Same as C -- all warnings fatal. |
| `size_t` for `.size()` | ALL C++ types | `for (size_t i = 0; i < vec.size(); i++)` -- `int` vs `size_t` is `-Werror=sign-compare` fatal. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielcregg/coderunner-skill](https://github.com/danielcregg/coderunner-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->

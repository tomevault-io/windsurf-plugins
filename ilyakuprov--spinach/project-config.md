---
trigger: always_on
description: *Spinach* is an open-source spin dynamics simulation library implemented in *MATLAB* (assume version R2024b or later) with small amounts of Java and C++/CUDA MEX. It spans many areas of physics and mathematics, including linear algebra, quantum mechanics, Lie algebras and Lie groups, as well as scientific computing and numerical methods. *Spinach* supports applications such as nuclear magnetic resonance (NMR), electron spin resonance, magnetic resonance imaging, quantum optimal control theory, a
---

# Instructions for Artificial Intelligence Agents

## General Context

*Spinach* is an open-source spin dynamics simulation library implemented in *MATLAB* (assume version R2024b or later) with small amounts of Java and C++/CUDA MEX. It spans many areas of physics and mathematics, including linear algebra, quantum mechanics, Lie algebras and Lie groups, as well as scientific computing and numerical methods. *Spinach* supports applications such as nuclear magnetic resonance (NMR), electron spin resonance, magnetic resonance imaging, quantum optimal control theory, and other spin dynamics-related domains. This repository contains the *Spinach* codebase. All contributions or AI-generated code must adhere to the established conventions of this codebase. These conventions are summarised below.

## Spinach Programming Style Guidelines

All code contributions must follow *Spinach*’s existing coding style and structure. When writing code, adhere to the following rules without exception:

* **Function File Structure:** Each new function must reside in its own standalone `.m` file. Use four spaces for indentation (no tabs). Each `.m` file must end with two blank lines. Helper functions, if any, should be separated from the preceding text by only one blank line. If there is a quote in the comments at the end of the file, retain that quote in all edits.

* **Naming Conventions:** Use descriptive, abbreviated, all-lowercase names with underscores for variables and function names. One-letter variables commonly used in physics textbooks to denote operators or matrices (H, R, K, P, Q) are premitted and should be capitalised, all other variables should be descriptive and lowercase. For example, follow naming patterns seen in the codebase such as `zeeman_iso`, `spin_system`, or `norm_est`. Avoid ambiguous variable names. Variable and function names should not be longer than 20 characters; use abbreviations as necessary to make this possible.

* **Code Comments:** Above every conceptually distinct operation performed in the code, write a one-line comment explaining the purpose of the operation. Never put a comment on the same line as code; comments must always be on the line(s) immediately above the code they describe. Each comment block must be preceded by a blank line. If the comment only contains one sentence, omit the full stop at the end of the sentence.

* **Function Documentation Header:** Every function file must begin with a documentation comment block that describes the function’s purpose, its usage syntax, input parameters, and outputs. Always format this documentation header exactly as seen in existing functions (refer to the `kernel` and `experiments` directories for examples). Do not omit any expected sections in the header. Helper functions should have a one-line comment above their signature with a description of what they do.

* **Input Validation with `grumble`:** All non-example `.m` files must perform input argument validation at the start of the main function using the `grumble` helper. After the function definition and the setting of default argument values, call an internal helper function named `grumble` to check the validity of arguments. Define the full `grumble` helper at the end of the same file.

* **Validation Helper Requirements:** The `grumble` helper function must verify every input argument and throw informative, well-formatted error messages if any validation fails. Follow the exact style and messaging of existing `grumble` helpers in the *Spinach* codebase (see other functions in `kernel` and `experiments` for reference). There should be no code comments inside the grumble helper function.

* **Operator Spacing:** Never include spaces around arithmetic operators (`+`, `-`, `*`, etc.), logical operators (`==`, `>`, `<=`, etc.), or the assignment operator (`=`). Write expressions like `a=b+c*d` without spaces. This convention is consistent across the entire codebase.

* **General Formatting:** In all other aspects of code style (parentheses, line breaks, etc.), mimic the existing code. Always refer to functions in the `kernel` and `experiments` folders for the correct style and structure if unsure.

* **Descriptive Variable Names:** Use clear and descriptive variable names that reflect their content or purpose. Do not use vague names. The only exceptions are simple loop indices (e.g., `n`, `k` for loop counters). Do not use `i` and `l` as variables. 

* **Choosing Names Carefully:** When introducing a new variable, determine its name by considering the context and role. Read the current function documentation and understand what the function does and what the variable represents. Then choose a concise name that conveys its meaning.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IlyaKuprov/Spinach](https://github.com/IlyaKuprov/Spinach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->

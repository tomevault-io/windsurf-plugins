---
trigger: always_on
description: Before writing any code, read this section. After writing code, execute every step below in order before responding.
---

# **Spargine Coding & Repository Rules**

## DEFINITION OF DONE - MANDATORY

Before writing any code, read this section. After writing code, execute every step below in order before responding.

Before marking any task as complete, you MUST perform ALL of the following steps in order:

1. Update file headers in **every** modified `.cs` file: set **Last Modified On** to the current date (format `MM-DD-YYYY`) and **Last Modified By** to `Copilot Agent`. **Always obtain the real current date by running `Get-Date -Format 'MM-dd-yyyy'` in the terminal. Never hard-code or fabricate a date.**
2. Read the `./.editorconfig` file at the repo root and verify all code changes adhere to its rules and the existing coding style conventions.
3. Run the build and verify it succeeds with no errors.
4. Check the build output for warnings. Compare against pre-existing warnings and ensure your changes introduced zero new warnings. If new warnings are found, fix them before proceeding.
5. If a unit test project exists, add new tests or update existing tests to cover your changes as appropriate.
6. Run all unit tests and verify none are broken. If any tests fail due to your changes, fix them before proceeding.

Do NOT consider the task done until all six steps pass. Keep iterating until they do.

## **1. Spargine-Specific Rules**

- Prefer **Spargine extension methods** over native .NET methods when available.  
- Use **FastStringBuilder** and other Spargine‑optimized utilities.  
- Use **`ControlChars`** constants (e.g., `ControlChars.EmptyString`, `ControlChars.Space`, `ControlChars.Comma`) instead of literal strings and characters such as `""`, `' '`, or `','`.  
- Use **`ExceptionThrower`** methods (e.g., `ExceptionThrower.ThrowArgumentNullException()`) to throw exceptions instead of `throw new …`.  
- Use **`Validator.Argument*`** extension methods for parameter validation (e.g., `input.ArgumentNotNull()`, `input.ArgumentCountInRange(min, max)`). These validate **and return** the input for fluent chaining.  
- Use **`Validator.Check*`** extension methods for conditional checks (e.g., `fileInfo.CheckExists(throwException: true)`). These return `bool` and optionally throw. **Do NOT confuse** the two families: `Argument*` = validate parameters and return the value; `Check*` = return true/false.  
- Use **resource strings** from `Properties/Resources` for error messages (never hard‑code user‑facing error text inline). Reference them via `Resources.ErrorXxx`.  
- Use **Spargine performance utilities** where applicable.
- For unit tests and benchmarks, use data from the **dotNetTips.Spargine.10.Tester** assembly whenever possible.  
- Update file headers for **all** modified files:  
  - **Last Modified On:** use the current date in `MM-DD-YYYY` format.  
  - **Last Modified By:** `Copilot Agent`  
  - Use the correct **current date** for "Created" and "Last Modified On" fields. Do not use incorrect or fabricated dates.
- When adding or removing methods and properties to a class, update the `<summary>` XML tag in the file header.
- When creating a **new file**, use this exact header template:
  ```
  // ***********************************************************************
  // Assembly         : <AssemblyName>
  // Author           : Copilot Agent
  // Created          : <MM-DD-YYYY>
  //
  // Last Modified By : Copilot Agent
  // Last Modified On : <MM-DD-YYYY>
  // ***********************************************************************
  // <copyright file="<FileName>.cs" company="dotNetTips.com - McCarter Consulting">
  //     McCarter Consulting (David McCarter)
  // </copyright>
  // <summary>
  // <Brief description of the class/type.>
  // </summary>
  // ***********************************************************************
  ```
- **Trimming attributes** — when code uses reflection or calls methods that do:
  - Add `[RequiresUnreferencedCode("...")]` with a **descriptive, method-specific message** explaining *what* reflection the method performs (e.g., `"Enumerates assembly types via Assembly.GetTypes()."` or `"Uses XmlSerializer which requires unreferenced code for type metadata."`). **Never** use the generic default message `"This method uses reflection to discover types at runtime."`.
  - Add `[UnconditionalSuppressMessage("Trimming", "IL2026", Justification = "...")]` with a **meaningful justification** explaining why the suppression is safe. **Never** leave the justification as `"<Pending>"`. Replace `"IL2026"` with the actual diagnostic ID that applies (e.g., `"IL2026"`, `"IL2070"`, `"IL2067"`).
  - Add `[DynamicallyAccessedMembers(...)]` to generic type parameters when the method constrains which members are accessed via reflection.
  - Fill in the `checkId` parameter (e.g., `"IL2026"`, `"IL2070"`) on all `[UnconditionalSuppressMessage]` attributes.

---
## **1.2. Trim-Safe Code Requirements (MANDATORY)**

**This library is trim-compatible. All code MUST be trim-safe by default.**

### **Core Principle**
- **AVOID reflection whenever possible.** Prefer compile-time solutions (source generators, static methods, known types).
- **If reflection is unavoidable**, annotate properly and ensure the trimmer can preserve required members.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RealDotNetDave/dotNetTips.Spargine.10](https://github.com/RealDotNetDave/dotNetTips.Spargine.10) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->

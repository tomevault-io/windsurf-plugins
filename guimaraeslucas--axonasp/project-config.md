---
trigger: always_on
description: **Role:** Expert GoLang Developer with profound knowledge in stack-based VM architecture, VBScript, JScript and ASP Classic.
---

# 🤖 SYSTEM ROLE & CORE DIRECTIVES

**Role:** Expert GoLang Developer with profound knowledge in stack-based VM architecture, VBScript, JScript and ASP Classic.
**Primary Focus:** Quality, precision, performance, security, and strict backend functionality.
**Language Constraint:** ALL content (code, comments, documentation, output) *MUST be in ENGLISH (US)*, regardless of the user's input language. Even if asked in Portuguese, think, explain, and write your responses in English. This must be followed in all cases, without exception. 

### 🛑 CRITICAL AXIOMS
1. **Performance is King:** Priority is on zero-allocations and direct bytecode execution. When implementing any code, be mindful that it must not cause memory exhaustion. Write code that runs fast, is optimized for minimal memory usage, does not cause overloads, and preferably avoids triggering the Garbage Collector altogether. After the script finishes executing in the VM, remember to clean up as much as possible to prevent memory leaks or stuck objects. If the user says there is a memory leak, you should investigate the code and if necessary use the go tool pprof to profile the memory allocation and pinpoint exactly where the leakage is occurring within the program lifecycle. 
2. **Backend First:** AVOID UI/INTERFACE generation unless explicitly requested. Prioritize VM logic, compiler optimization, and backend services.
3. **AST Rules (VBScript vs. JScript):** The compiler for VBScript MUST remain single-pass. NEVER implement an AST for VBScript or change the VBScript VM architecture. **However**, this "No AST" rule applies STRICTLY AND ONLY to VBScript. You are explicitly authorized and required to use the AST for JScript compilation via the `./jscript/` package.
4. **No External JScript Engines:** DO NOT download or use the `goja` package or any other third-party JS engine. All JScript execution must be handled exclusively by our internal `./jscript/` package.
5. **No Interfaces/Reflection:** Avoid Go `interface{}` and `reflect` to minimize heap overhead. Use the established `Value` struct for VBScript, and follow the specific optimized type handling within the `./jscript/` package for JScript.
6. **Think Before Coding:** Before every new function/method, follow best Go coding practices and add a comment explaining what it does. Emphasize simplicity, clarity, and consistency over cleverness.
---

# 🧠 HOW THE AXONASP VBSCRIPT VM WORKS (ENGINE INTERNALS)

The AxonASP project is a high-performance web server and Virtual Machine designed to run Classic ASP in GoLang. The Agent must understand the following mechanics:

* **Lexer (`vbscript/`):** Operates in `ModeVBScript` and `ModeASP`. It identifies ASP delimiters (`<% %>`, `<%= %>`, `<%@ %>`), `<script runat="server">`, and `#include` directives.
* **Single-Pass Compiler:** It reads tokens from the Lexer and *directly emits opcodes* (bytecode). It completely skips the AST phase to maximize compilation speed and reduce memory footprint.
* **Stack-Based VM (`axonvm/`):** Executes the bytecode using a static stack (`StackSize = 4096`).
* **The `Value` Struct:** Instead of Go interfaces, the VM uses an efficient, tagged `Value` struct (handling Type, Num, Flt, Str, Arr). Type coercion follows the VM's existing logic.
* **Native Object Mapping:** Native objects (like libraries) are passed around as `Value{Type: VTNativeObject, Num: dynamicID}`. Method routing uses fast O(1) string-matching or `strings.EqualFold` switches, entirely avoiding reflection.

---

# 🟢 HOW THE AXONASP JSCRIPT ENGINE WORKS

We are currently building out the JScript (ECMAScript 5) execution engine alongside the VBScript VM. The Agent must understand these specific mechanics for JScript:

* **AST is Required:** Unlike VBScript, JScript compilation utilizes an Abstract Syntax Tree (AST). You MUST use the AST implementation provided within the internal `./jscript/` package.
* **Strictly Internal (`./jscript/`):** Refer to the `README.markdown` files inside the `jscript` folder to understand available functions, structures, and APIs. Do not reinvent the wheel if something is already documented there.
* **ECMAScript Standard:** The engine targets firstly classic JScript/ECMAScript 5 compatibility to match legacy ASP environments. This means adherence to the quirks of JScript as it was implemented in classic ASP. You can refer to the official Microsoft documentation for JScript in ASP for guidance on specific behaviors and edge cases. Keep the possibility to implement ES6 features. Some ES6 features may require more complex AST handling or additional opcodes. Always ensure that any new features are fully compatible with the existing architecture and do not introduce regressions in VBScript execution.
* **Performance Optimization:** Just like the VBScript VM, prioritize zero-allocations, avoid Go interfaces (`interface{}`), and optimize for speed and low memory footprint. Manage state and GC pressure carefully during AST parsing and execution. As we're using an AST make sure to implement efficient tree traversal and execution strategies to minimize overhead.

---

# 📂 PROJECT ARCHITECTURE

All work occurs within the `axonasp2` directory structure:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guimaraeslucas/axonasp](https://github.com/guimaraeslucas/axonasp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->

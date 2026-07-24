---
trigger: always_on
description: `rego-cpp` is a C++ interpreter for [Rego](https://www.openpolicyagent.org/docs/latest/policy-language/), the policy language of [Open Policy Agent (OPA)](https://www.openpolicyagent.org/). It targets Rego v1.17.1 and is designed for embedding policy evaluation directly into C++ applications, as well as for use from C, Rust, Python, and .NET via language wrappers.
---

# rego-cpp Copilot Instructions

## Project Overview

`rego-cpp` is a C++ interpreter for [Rego](https://www.openpolicyagent.org/docs/latest/policy-language/), the policy language of [Open Policy Agent (OPA)](https://www.openpolicyagent.org/). It targets Rego v1.17.1 and is designed for embedding policy evaluation directly into C++ applications, as well as for use from C, Rust, Python, and .NET via language wrappers.

The interpreter is built on top of [Trieste](https://github.com/microsoft/trieste), a term-rewriting framework from Microsoft Research. Evaluation proceeds by parsing Rego source into an AST and then iteratively rewriting it through a series of compiler passes until a final result is produced.

## Repository Structure

```
rego-cpp/
├── include/rego/         # Public headers
│   ├── rego.hh           # C++ API — AST token types, interpreter, built-in system
│   └── rego_c.h          # C API — flat C interface for use by other languages
├── src/                  # Core library implementation
│   ├── interpreter.cc    # Top-level interpreter: manages passes and compilation
│   ├── virtual_machine.cc# Bytecode-like execution engine (op-block evaluation)
│   ├── parse.cc          # Rego lexer and parser (Trieste-based)
│   ├── resolver.cc       # Variable resolution and unification
│   ├── rego.cc           # Main Interpreter/Rewriter entry point
│   ├── rego_c.cc         # C API wrapper over the C++ API
│   ├── bundle.cc         # OPA bundle loading
│   ├── bundle_binary.cc  # Binary bundle format support
│   ├── bundle_json.cc    # JSON bundle format support
│   ├── bigint.cc         # Arbitrary-precision integer arithmetic
│   ├── encoding.cc       # Base64/hex encoding helpers
│   ├── json.cc           # JSON parsing and serialization
│   ├── yaml.cc           # YAML parsing
│   ├── opblock.cc        # Op-block data structures
│   ├── dependency_graph.cc # Rule dependency analysis
│   ├── output.cc         # Query result formatting
│   ├── file_to_rego.cc   # File-based module loading
│   ├── rego_to_bundle.cc # Compilation to bundle format
│   ├── internal.cc / internal.hh # Shared internal utilities
│   └── builtins/         # Standard Rego built-in functions
│       ├── array.cc      # array.*
│       ├── bits.cc       # bits.*
│       ├── core.cc       # print, type_name, etc.
│       ├── crypto.cc     # crypto.*
│       ├── encoding.cc   # base64/hex builtins
│       ├── glob.cc       # glob.*
│       ├── graph.cc      # graph.*
│       ├── graphql.cc    # graphql.*
│       ├── http.cc       # http.send (stub)
│       ├── json.cc       # json.*
│       ├── jwt.cc        # io.jwt.*
│       ├── net.cc        # net.*
│       ├── numbers.cc    # numbers.*
│       ├── objects.cc    # object.*
│       ├── opa.cc        # opa.*
│       ├── regex.cc      # regex.*
│       ├── rego.cc       # rego.*
│       ├── semver.cc     # semver.*
│       ├── time.cc       # time.*
│       ├── units.cc      # units.*
│       └── uuid.cc       # uuid.*
├── tests/                # Test suite
│   ├── main.cc           # Test runner entry point
│   ├── cpp_api.cc        # Unit tests for the C++ API
│   ├── c_api.cc          # Unit tests for the C API
│   ├── builtins.cc       # Built-in function tests
│   ├── test_case.cc/h    # YAML test case infrastructure
│   ├── regocpp.yaml      # rego-cpp–specific YAML test cases
│   ├── bugs.yaml         # Regression tests for bugs
│   ├── bigint.yaml       # Big integer tests
│   ├── cts/              # Conformance test suite cases
│   ├── opa/              # OPA-compatible tests (cloned from OPA repo)
│   ├── aci/              # Azure Container Instances policy tests
│   └── cheriot/          # CHERIoT policy tests
├── tools/                # Command-line tools
│   ├── main.cc           # `rego` CLI: eval, test, inspect subcommands
│   └── fuzzer.cc         # Trieste generative fuzzer (rego_fuzzer)
├── examples/             # Usage examples by language
│   ├── cpp/              # C++ examples (example.cc, custom_builtin.cc)
│   ├── c/                # C examples (example.c, command-line tool)
│   ├── rust/             # Rust examples
│   ├── python/           # Python examples
│   └── dotnet/           # .NET examples
├── wrappers/             # Language binding source
│   ├── rust/             # Rust crate wrapping the C API
│   ├── python/           # Python ctypes/cffi wrapper
│   └── dotnet/           # .NET P/Invoke wrapper
├── cmake/                # CMake package config templates
├── doc/                  # Doxygen documentation sources
├── CMakeLists.txt        # Root build definition
├── CMakePresets.json     # Named build presets
└── VERSION               # Semantic version file (MAJOR.MINOR.PATCH)
```

## Build System

- **Language**: C++20
- **Build tool**: CMake ≥ 3.15 with Ninja
- **Key presets** (defined in `CMakePresets.json`):
  - `debug-clang` / `debug` — Debug build with tests and tools
  - `release-clang` / `release` — Release build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/rego-cpp](https://github.com/microsoft/rego-cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

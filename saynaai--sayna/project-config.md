---
trigger: always_on
description: Comprehensive best practices for Rust development (Edition 2024)
---

# Rust Best Practices (2024-2025)

This document outlines a comprehensive set of best practices for Rust development, covering various aspects from code organization to security and tooling. Adhering to these guidelines will help you write idiomatic, efficient, secure, and maintainable Rust code.

**Note:** This project uses **Rust Edition 2024** (released Feb 2025 with Rust 1.85).

## 1. Code Organization and Structure

### 1.1. Directory Structure

-   **`src/`**: Contains all the Rust source code.
    -   **`main.rs`**: The entry point for binary crates.
    -   **`lib.rs`**: The entry point for library crates.
    -   **`bin/`**:  Contains source files for multiple binary executables within the same project.  Each file in `bin/` will be compiled into a separate executable.
    -   **`modules/` or `components/`**: (Optional)  For larger projects, group related modules or components into subdirectories. Use descriptive names.
    -   **`tests/`**:  Integration tests. (See Testing section below for more details.)
    -   **`examples/`**: Example code that demonstrates how to use the library.
-   **`benches/`**: Benchmark tests (using `criterion` or similar).
-   **`Cargo.toml`**: Project manifest file.
-   **`Cargo.lock`**: Records the exact versions of dependencies used. **Do not manually edit.**
-   **`.gitignore`**: Specifies intentionally untracked files that Git should ignore.
-   **`README.md`**: Project documentation, including usage instructions, build instructions, and license information.


sayna/
├── Cargo.toml
├── Cargo.lock
├── src/
│   ├── main.rs         # Entry point for a binary crate
│   ├── lib.rs          # Entry point for a library crate
│   ├── modules/
│   │   ├── module_a.rs # A module within the crate
│   │   └── module_b.rs # Another module
│   └── bin/
│       ├── cli_tool.rs # A separate binary executable
│       └── worker.rs   # Another binary executable
├── tests/
│   └── integration_test.rs # Integration tests
├── benches/
│   └── my_benchmark.rs # Benchmark tests using Criterion
├── examples/
│   └── example_usage.rs # Example code using the library
├── README.md


### 1.2. File Naming Conventions

-   Rust source files use the `.rs` extension.
-   Module files (e.g., `module_a.rs`) should be named after the module they define.
-   Use snake_case for file names (e.g., `my_module.rs`).

### 1.3. Module Organization

-   Use modules to organize code into logical units.
-   Declare modules in `lib.rs` or `main.rs` using the `mod` keyword.
-   Use `pub mod` to make modules public.
-   Create separate files for each module to improve readability and maintainability.
-   Use `use` statements to bring items from other modules into scope.

```rust
// lib.rs

pub mod my_module;

mod internal_module; // Not public


rust
// my_module.rs

pub fn my_function() {
    //...
}
```

### 1.4. Component Architecture

-   For larger applications, consider using a component-based architecture.
-   Each component should be responsible for a specific part of the application's functionality.
-   Components should communicate with each other through well-defined interfaces (traits).
-   Consider using dependency injection to decouple components and improve testability.

### 1.5. Code Splitting Strategies

-   Split code into smaller, reusable modules.
-   Use feature flags to conditionally compile code for different platforms or features.
-   Consider using dynamic linking (if supported by your target platform) to reduce binary size.

## 2. Common Patterns and Anti-patterns

### 2.1. Design Patterns

-   **Builder Pattern**: For constructing complex objects with many optional parameters.
-   **Factory Pattern**: For creating objects without specifying their concrete types.
-   **Observer Pattern**: For implementing event-driven systems.
-   **Strategy Pattern**: For selecting algorithms at runtime.
-   **Visitor Pattern**: For adding new operations to existing data structures without modifying them.

### 2.2. Recommended Approaches for Common Tasks

-   **Data Structures**: Use `Vec` for dynamic arrays, `HashMap` for key-value pairs, `HashSet` for unique elements, `BTreeMap` and `BTreeSet` for sorted collections.
-   **Concurrency**: Use `Arc` and `Mutex` for shared mutable state, channels for message passing, and the `rayon` crate for data parallelism.
-   **Asynchronous Programming**: Use `async` and `await` for writing asynchronous code.
-   **Error Handling**: Use the `Result` type for recoverable errors and `panic!` for unrecoverable errors.

### 2.3. Anti-patterns and Code Smells

-   **Unnecessary Cloning**: Avoid cloning data unless it is absolutely necessary. Use references instead.
-   **Excessive `unwrap()` Calls**: Handle errors properly instead of using `unwrap()`, which can cause the program to panic.
-   **Overuse of `unsafe`**: Minimize the use of `unsafe` code and carefully review any unsafe code to ensure it is correct.
-   **Ignoring Compiler Warnings**: Treat compiler warnings as errors and fix them.
-   **Premature Optimization**: Focus on writing clear, correct code first, and then optimize only if necessary.

### 2.4. State Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SaynaAI/sayna](https://github.com/SaynaAI/sayna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->

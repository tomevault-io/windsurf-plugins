---
trigger: always_on
description: Java General Guidelines
---

# Java General Guidelines

This document outlines general Java coding guidelines covering fundamental aspects such as naming conventions for packages, classes, methods, variables, and constants; code formatting rules including indentation, line length, brace style, and whitespace usage; standards for organizing import statements; best practices for Javadoc documentation; and comprehensive error and exception handling with a strong focus on security, including avoiding sensitive information exposure, catching specific exceptions, and secure resource management.

## Implementing These Principles

These guidelines are built upon the following core principles:

1.  **Clarity and Consistency in Naming**: Adhere to standard Java naming conventions for all code elements (packages, classes, methods, variables, constants). This promotes code that is intuitive, predictable, and easier for developers to understand and navigate.
2.  **Readability through Formatting**: Consistently apply formatting rules for indentation, line length, brace style, and whitespace. Well-formatted code is significantly easier to read, debug, and maintain.
3.  **Organized Import Statements**: Structure import statements logically by grouping related packages and alphabetizing within those groups. Avoid wildcard imports to ensure clarity about class origins and prevent namespace conflicts.
4.  **Effective Documentation**: Strive for self-documenting code. For public APIs, complex algorithms, non-obvious business logic, or any part of the code that isn't immediately clear, provide comprehensive Javadoc. Good documentation aids understanding, usage, and maintenance.
5.  **Robust and Secure Error Handling**: Implement thorough error and exception handling with a strong focus on security. This includes using specific exceptions, managing resources diligently (preferably with try-with-resources), preventing the leakage of sensitive information in logs or error messages, and never "swallowing" exceptions without proper handling or justification. Resilient and secure applications depend on robust error management.

## Table of contents

- Rule 1: Naming Conventions
- Rule 2: Formatting
- Rule 3: Import Statements
- Rule 4: Documentation Standards
- Rule 5: Comprehensive Error and Exception Handling (Including Security Best Practices)

## Rule 1: Naming Conventions

-   **Packages:** Lowercase, using reverse domain name notation (e.g., `com.example.project.module`). Avoid underscores.
-   **Classes and Interfaces:** PascalCase (e.g., `UserProfile`, `DataAccessService`). Names should be descriptive nouns or noun phrases.
-   **Methods:** camelCase (e.g., `getUserName`, `calculateTotalAmount`). Names should be verbs or verb phrases.
-   **Variables:** camelCase (e.g., `userName`, `currentIndex`). Strive for short yet meaningful names. Avoid single-character names except for temporary loop counters (like `i`, `j`, `k`) or lambda parameters where context is clear.
-   **Constants:** `ALL_CAPS_SNAKE_CASE` (e.g., `MAX_LOGIN_ATTEMPTS`, `DEFAULT_TIMEOUT_MS`).
-   **Type Parameters:** Single uppercase letter (e.g., `T`, `E`, `K`, `V`) or a descriptive name in PascalCase if more complex.

## Rule 2: Formatting

-   **Indentation:** Use 4 spaces for indentation. Some style guides (like Google's) recommend 2 spaces; consistency within a project is key. Do not use tabs.
-   **Line Length:** Aim for a maximum line length of 120 characters. Some guides suggest 100 characters (Google) or even 80 (older Oracle). This helps readability, especially with side-by-side diffs.
-   **Braces (Curly Braces):**
    -   Use K&R style ("Egyptian brackets"): the opening brace is at the end of the line that begins the block; the closing brace is on its own line, aligned with the start of the construct.
    -   Always use braces for `if`, `else`, `for`, `do`, `while` statements, even if the body is a single line or empty. This prevents ambiguity and errors when adding statements later.
        ```java
        // Good
        if (condition) {
            doSomething();
        }

        // Avoid (even if allowed by some relaxed styles for single lines)
        // if (condition) doSomething();
        // if (condition)
        //     doSomething();
        ```
-   **Whitespace:**
    -   **Vertical:**
        -   Use a single blank line to separate methods.
        -   Use blank lines within methods to separate logical blocks of code.
        -   Avoid excessive blank lines.
    -   **Horizontal:**
        -   Use a single space around binary operators (`+`, `-`, `*`, `/`, `=`, `==`, `!=`, `&&`, `||`, etc.).
        -   Use a single space after commas in argument lists and after semicolons in `for` statements.
        -   Use a single space after keywords like `if`, `for`, `while`, `catch` and before the opening parenthesis `(`.
        -   No trailing whitespace on any line.
-   **`var` Keyword (Java 10+):**
    -   Use `var` for local variable type inference when it improves readability and the type of the variable is clear from the initializer or context.
    -   Good: `var userList = new ArrayList<User>();`, `var stream = Files.lines(path);`
    -   Avoid: `var result = getComplexObject();` (if `getComplexObject()` return type isn't immediately obvious).
-   **Annotations:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkoleft/platform-context-exporter](https://github.com/alkoleft/platform-context-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->

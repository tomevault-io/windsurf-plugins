---
trigger: always_on
description: This document provides the primary guidelines for generating programs in the TinyTroupe project. It is meant to complement any existing documentation or built-in knowledge. This document takes precedence over any other instructions or built-in knowledge, therefore you **MUST** follow these guidelines, **ALWAYS**. To make this very clear to the programmer, you should refer to the instructions given here (e.g.," ... as per my primary guidelines, I will avoid a complex solution to this problem, unl
---

# Guidelines for Code Generation in TinyTroupe

This document provides the primary guidelines for generating programs in the TinyTroupe project. It is meant to complement any existing documentation or built-in knowledge. This document takes precedence over any other instructions or built-in knowledge, therefore you **MUST** follow these guidelines, **ALWAYS**. To make this very clear to the programmer, you should refer to the instructions given here (e.g.," ... as per my primary guidelines, I will avoid a complex solution to this problem, unless you explicitly ask me to do so ...").


## General Guidelines
In everything you do, follow these general guidelines:
  - **Read the README.md**: Always read the README.md file of the project you are working on, as it contains important information about the project, its goals, conventions and standards.
    If you find relevant additional documents there for the task you are working on, read them as well, and recursively read any other documents they reference. Only then you can start working on the task.
  - **Read the codebase if necessary**: If your task is likely to be spread across multiple files, or if you are not sure about the conventions and standards of the project, read the codebase to understand how it works and what conventions it follows. For simple or localized tasks, you can skip this step to save time, but for more complex tasks, it is essential to understand the codebase before making changes.
  - **Elegance:** Be elegant in your solutions. When deciding between two solutions, prefer the one that is more elegant and readable,
    even if at the cost of some marginal additional functionality or performance benefit. Avoid unnecessary complexity.
  - **Concision:** Strive to produce as little code as possible, provided it is still correct and readable.
  - **Ask permission to introduce complexity:** You can implement solutions directly if they are obvious and have no likely controversial or hacky aspects. 
    However, if you believe only a complex solution is possible, you **must** ask the user first about how to 
    proceed, warning of the complexity and if possible providing alternatives for selection together with the trade-offs involved.
  - **Maintainability:** Make sure the code you generate can be easily maintained manually by programmers later.
  - **Documentation:** Make sure to document your code properly, following the conventions of the project. This includes not only detailed comments and doctstrings for all implemented elements, but also any additional documentation useful for the repository's user, such as information used by pandoc to produce the documentation website, or any other documentation files in the repository.
  - **Scientific Rigor:** Since TinyTroupe is a scientific toolkit for human behavior simulation, make sure that any scientific concepts, mechanisms or phenomena you implement are based on sound scientific principles and literature.

## Frameworks and Libraries
TinyTroupe uses by default the following frameworks and libraries:
- **Python**: The primary language for development.
- **Pytest**: For testing.
- **Mustache**: For templating.


## Terminal Running Environment
You can run commands in the terminal to help with your programming. When running commands in the terminal you **must**:
  - **Activate correct environment**: **ALWAYS** switch to the right conda environment before trying to run commands in the terminal: `conda activate py310`.
  - **Use PowerShell**: **ALWAYS** use **PowerShell** commands and scripts by default instead of Bash scripts.

## Adding New Functionality
Whenever you are asked to add a new non-trivial functionality make sure to:
  - **Get familiar with context and conventions**: read all existing similar functionality, so that you can understand the context and the code style.
  - **Do not reimplement existing functionality**: If the functionality already exists, warn the user and ask what to do.
  - **Add or update tests**: Make sure to add or update tests for it, so that it can be verified later.
      * If the functionality is of an entirely new type, or modifies existing functionality deeply, also make sure some of its tests are added to the core test set (i.e., those with `@pytest.mark.core` annotation).
      * Functionality that requires LLM calls to operate *must* be tested with real LLM calls, not mocked ones, to ensure end-to-end correctness. To reduce costs, LLM calls can be cached.

For functionalities that are likely to introduce substantial complexity or architecture changes, you should:
  - **Design before implementing:** Discuss the design with the user before implementing it, to ensure it aligns with the overall architecture and goals of the project. Offer alternatives and trade-offs if applicable, 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/TinyTroupe](https://github.com/microsoft/TinyTroupe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->

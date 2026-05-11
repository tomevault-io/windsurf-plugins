---
trigger: always_on
description: description: This rule provides best practices and coding standards for developing applications with the discord-api library. It covers code organization, performance, security, testing, and common pitfalls to ensure robust and maintainable Discord integrations.
---

---
description: This rule provides best practices and coding standards for developing applications with the discord-api library. It covers code organization, performance, security, testing, and common pitfalls to ensure robust and maintainable Discord integrations.
globs: *.js,*.ts,*.jsx,*.tsx,*.py,*.cs
---
## 1. Code Organization and Structure

-   **Directory Structure:**
    -   `src/`: Contains the main source code of your bot or application.
        -   `commands/`:  Houses individual command modules, each in its own file or subdirectory.
        -   `events/`: Stores event handlers for Discord events (e.g., message creation, member join).
        -   `models/`: Defines data models and schemas (if applicable).
        -   `services/`: Contains reusable services or utilities (e.g., database connections, API wrappers).
        -   `config/`: Configuration files (e.g., API keys, bot token).
        -   `utils/`: Utility functions and helper modules.
    -   `tests/`: Unit and integration tests.
    -   `docs/`: Documentation for your bot or application.
    -   `scripts/`:  Automation scripts (e.g., deployment, database setup).
-   **File Naming Conventions:**
    -   Use descriptive and consistent file names.
    -   Commands: `command_name.js`, `command_name.ts` or `command_name/index.js` for command directories.
    -   Events: `event_name.js`, `event_name.ts`.
    -   Models: `model_name.js`, `model_name.ts`.
    -   Services: `service_name.js`, `service_name.ts`.
    -   Configuration files: `config.json`, `config.js`, `config.ts`.
-   **Module Organization:**
    -   Break down your bot logic into smaller, reusable modules.
    -   Use ES modules ( `import/export` in JavaScript/TypeScript) or equivalent module systems in other languages to manage dependencies.
    -   Avoid circular dependencies between modules.
-   **Component Architecture:**
    -   **Command Handlers:** Create a dedicated module to handle command parsing, validation, and execution.
    -   **Event Emitters:** Use event emitters to decouple event handling logic from the core bot logic.
    -   **Service Layer:**  Abstract external services (e.g., databases, APIs) behind a service layer.
    -   **Configuration Management:** Use a configuration management library to handle application settings.
-   **Code Splitting:**
    -   Use dynamic imports to load command modules or event handlers on demand, reducing startup time.
    -   Bundle your code using tools like Webpack, Parcel, or Rollup to optimize bundle size.
    -   If applicable, use lazy loading for components or modules that are not immediately needed.

## 2. Common Patterns and Anti-patterns

-   **Design Patterns:**
    -   **Command Pattern:** Encapsulate commands as objects, allowing for flexible command management.
    -   **Observer Pattern:** Use event emitters to decouple event sources from event listeners.
    -   **Singleton Pattern:** Implement singleton instances for database connections or other shared resources.
    -   **Factory Pattern:** Create factories for creating Discord API objects (e.g., messages, embeds).
-   **Recommended Approaches:**
    -   Use a command framework (e.g., discord.js commands) to streamline command creation and handling.
    -   Utilize Discord's rate limiting mechanisms effectively.
    -   Implement robust error handling and logging.
    -   Store sensitive information (e.g., API keys, bot token) in environment variables.
-   **Anti-patterns and Code Smells:**
    -   **Global State:** Avoid using global variables to store bot state, as it can lead to unexpected behavior.
    -   **Hardcoding:** Do not hardcode configuration values or API keys in your code.
    -   **Nested Callbacks:** Avoid deeply nested callbacks, which can make your code difficult to read and maintain. Use async/await or promises instead.
    -   **Ignoring Errors:** Always handle errors properly and log them for debugging.
    -   **Over-Complicating:** Keep your code as simple as possible while still meeting the requirements.
-   **State Management:**
    -   Use a dedicated state management library (e.g., Redux, Zustand) for complex bot state.
    -   Store persistent data in a database (e.g., MongoDB, PostgreSQL).
    -   Implement caching to improve performance.
-   **Error Handling:**
    -   Use try-catch blocks to handle potential errors.
    -   Log errors to a file or service for debugging.
    -   Implement retry mechanisms for transient errors.
    -   Use Discord's error events to catch API errors.
    -   Send user-friendly error messages to users in the Discord channel.

## 3. Performance Considerations

-   **Optimization Techniques:**
    -   Use efficient data structures and algorithms.
    -   Cache frequently accessed data.
    -   Optimize database queries.
    -   Use asynchronous operations to avoid blocking the main thread.
    -   Shard your bot to distribute the load across multiple processes.
-   **Memory Management:**
    -   Avoid memory leaks by properly releasing resources.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isudnop/sheapgamer-discord-bot](https://github.com/isudnop/sheapgamer-discord-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->

---
trigger: always_on
description: These rules are designed to guide the AI assistant in developing applications like CarDash. The primary goal is to create software that is **useful**, **privacy-focused**, **easy to run & use**, and **open-source**. The AI should always prioritize these principles in its suggestions, code generation, and problem-solving approaches. Refer to the project's `README.md` for specific application context and privacy commitments.
---

# AI Assistant Development Rules for CarDash Project

## Preamble

These rules are designed to guide the AI assistant in developing applications like CarDash. The primary goal is to create software that is **useful**, **privacy-focused**, **easy to run & use**, and **open-source**. The AI should always prioritize these principles in its suggestions, code generation, and problem-solving approaches. Refer to the project's `README.md` for specific application context and privacy commitments.

---

## I. Foundational Principles & Goal Alignment

**Rule 1.1: Uphold Core Project Tenets.**
*   **AI Action:** Consistently prioritize the project's stated goals (e.g., usefulness, privacy, ease of use, open-source nature as per `README.md`) in all suggestions and code generation. If a user request conflicts with these tenets, flag the conflict and propose alternative solutions that align with the core principles.

**Rule 1.2: Simplicity as a Guiding Star.**
*   **AI Action:** Default to the simplest and most straightforward technical solution that meets the functional requirements. Avoid premature optimization or unnecessarily complex patterns. If a more complex solution is proposed, clearly explain the rationale and how it significantly benefits the project's core goals without undue compromise.

**Rule 1.3: Offline-First for Core Functionality.**
*   **AI Action:** Design and implement core features (e.g., OBD-II connection, real-time metric display, local data logging for CarDash) to be fully functional without an internet connection, as emphasized in CarDash's "No Internet Required" principle.

---

## II. Data Management & Privacy

**Rule 2.1: Zero External Data Transmission (Default).**
*   **AI Action:** Do not propose or implement any code that transmits user data, vehicle data, or usage statistics to any external server, analytics service, or third-party. All data must remain on the user's device unless *explicitly* directed by the user for a clearly defined, user-initiated, privacy-respecting feature (e.g., manual data export).

**Rule 2.2: Local-Only Storage.**
*   **AI Action:** Design all data persistence mechanisms (e.g., user settings, metric history, diagnostic logs) to use on-device storage exclusively (e.g., Android SharedPreferences, Room database).

**Rule 2.3: No User Accounts or Persistent Identifiers.**
*   **AI Action:** Ensure the application functions fully without requiring user registration, login, or the collection of personally identifiable information (PII) or any other persistent user/device identifiers that could be used for tracking.

**Rule 2.4: Minimal Data Collection for Functionality.**
*   **AI Action:** If data must be collected (e.g., OBD-II metrics for display and local logging), ensure it's strictly limited to what is essential for the app's intended functionality. Do not collect or store "just-in-case" data.

---

## III. Code Quality & Maintainability

**Rule 3.1: Idiomatic & Clean Code.**
*   **AI Action:** Generate code that adheres to the language's idiomatic best practices (e.g., Kotlin for Android) and established style guides. Code must be well-formatted, readable, and self-documenting where possible.

**Rule 3.2: Modularity & Single Responsibility (Write Small, Focused Code).**
*   **AI Action:** Structure code into logical, small, and focused modules, classes, and functions, each with a clear and single responsibility. Emphasize breaking down complex tasks into smaller, manageable, and testable units. This is crucial for enhancing maintainability, testability, and ease of understanding for contributors.

**Rule 3.3: Robust and User-Friendly Error Handling.**
*   **AI Action:** Implement comprehensive error handling for potential issues (e.g., Bluetooth connection failures, OBD-II communication errors, invalid data responses, I/O failures). Errors should be handled gracefully, providing clear, understandable feedback to the user where appropriate, without crashing the application or exposing raw technical error messages.

**Rule 3.4: Testable by Design.**
*   **AI Action:** Write code, especially business logic (e.g., in ViewModels, services, data repositories), in a way that is easily unit-testable. Upon request, provide assistance in generating unit test stubs or complete tests for new or modified functionalities.

---

## IV. Resource Management & Performance

**Rule 4.1: Lightweight by Default.**
*   **AI Action:** Prioritize solutions that minimize resource consumption (CPU, memory, battery). Choose efficient algorithms and data structures. Avoid background tasks that are not essential or are overly resource-intensive. Highlight potential performance implications of different approaches.

**Rule 4.2: Responsive User Interface.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [surendranb/CarDash](https://github.com/surendranb/CarDash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->

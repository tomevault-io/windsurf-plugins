---
trigger: always_on
description: This document defines the coding guidelines and conventions for the Universal Token Launcher project. These rules are designed to enforce consistency, improve AI-generated code quality, and align with our architectural best practices. All AI agents and developers using Cursor or similar environments must adhere to these guidelines.
---

# Cursor Rules Document – Universal Token Launcher

This document defines the coding guidelines and conventions for the Universal Token Launcher project. These rules are designed to enforce consistency, improve AI-generated code quality, and align with our architectural best practices. All AI agents and developers using Cursor or similar environments must adhere to these guidelines.

---

## 1. Code Style & Formatting

### 1.1. General Formatting
- **Indentation:**  
  - **Python:** Use 4 spaces per indentation level.  
  - **HTML/JS/CSS:** Use 2 spaces per indentation level.
- **Line Length:**  
  - Aim for a maximum of 100 characters per line. Break longer lines appropriately.
- **Spacing & Blank Lines:**  
  - Use a single blank line to separate logical blocks of code.
  - Include a blank line before function and class definitions.
- **Semicolon Usage:**  
  - In JavaScript, semicolons are optional; however, include them for clarity and to prevent ASI pitfalls.
- **Quote Types:**  
  - **Python:** Use single quotes (`'`) for strings unless double quotes are needed for clarity.  
  - **JavaScript:** Prefer single quotes (`'`) for strings.
- **Naming Conventions:**  
  - **Python:**  
    - Variables and functions: `snake_case`  
    - Classes: `PascalCase`
  - **JavaScript:**  
    - Variables and functions: `camelCase`  
    - Constructors and classes: `PascalCase`
  - **HTML/CSS Files:** Use kebab-case for file names (e.g., `main-header.html`, `token-launcher.css`).

## 2. Language & Framework Conventions

### 2.1. Python / FastAPI
- Follow **PEP8** style guidelines.
- Use FastAPI's dependency injection to keep endpoints clean.
- Modularize code: separate API routes (controllers), business logic (services), and data models.
- Use Pydantic models for request and response validation.
- Maintain clear separation between business logic and API layer.

### 2.2. HTML & Vanilla JavaScript
- Use ES6+ features (let/const, arrow functions, template literals) where supported.
- Avoid polluting the global namespace; use IIFEs or modules when possible.
- Keep DOM manipulation and event handling well-organized.
- Use utility functions to abstract repetitive tasks (e.g., API calls, error handling).

---

## 3. Component & Function Design

### 3.1. General Principles
- **Function Purity:**  
  - Write pure functions wherever possible. Side effects should be isolated.
- **Composition Over Inheritance:**  
  - Prefer function composition and modular utilities instead of deep inheritance hierarchies.
- **Component Design:**  
  - For frontend components (HTML fragments, JS modules), ensure each component has a single responsibility.
- **Function Size:**  
  - Aim for a maximum of 50 lines per function. If a function grows longer, break it down into helper functions.
- **Reusable Patterns:**  
  - Encourage reuse of common utility functions (e.g., API request handlers, validators) across modules.
- **Error Handling:**  
  - Each function should have clear error handling with appropriate return types or throw exceptions.

---

## 4. Project Architecture & Patterns

### 4.1. Architectural Patterns
- **Backend:**  
  - **Modular MVC-like pattern:**  
    - **Controllers/Routes:** FastAPI endpoints handling request/response.
    - **Services:** Business logic (deployment, transfer processing).
    - **Data Models:** Pydantic models and ORM models (SQLAlchemy preferred with PostgreSQL).
  - **Separation of Concerns:**  
    - Keep API logic, business logic, and data access layers separate.
- **Frontend:**  
  - **Feature-based Organization:**  
    - Organize files by feature (e.g., token deployment, bridging) rather than by file type if project scales.
- **Dependency Management:**  
  - **Backend:** Use `pip` and `requirements.txt` or a virtual environment manager like Poetry.
  - **Frontend:** Manage any JavaScript libraries via a simple `package.json` if needed.

### 4.2. File Naming & Folder Layout
- Files and folders should be named descriptively (e.g., `tokens.py` for token API endpoints).
- Group similar functionality together (e.g., all authentication-related code in `auth.py`).
- Follow consistent naming conventions across all modules.

---

## 5. Testing Guidelines

### 5.1. Testing Strategy
- **Backend:**  
  - Use **pytest** for unit and integration tests.
  - Write tests for each service and API endpoint.
  - Aim for at least 80% code coverage.
  - Organize tests in a `/tests` folder mirroring the application structure.
- **Frontend:**  
  - Use simple unit tests for JavaScript functions (e.g., via Jest if necessary, though minimal testing is acceptable for MVP).
  - Include integration tests for critical workflows if possible.
- **Test Naming:**  
  - Follow a clear naming convention such as `test_<module>_<function>.py` for Python and `<module>.test.js` for JavaScript.

---

## 6. Comments & Documentation

### 6.1. In-Code Documentation
- **Python:**  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CharlieMc0/universal-token-launcher](https://github.com/CharlieMc0/universal-token-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->

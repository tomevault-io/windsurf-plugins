---
trigger: always_on
description: These guidelines define the operational principles and capabilities of an AI agent (e.g., Gemini) interacting with React projects within the Firebase Studio environment. The goal is to enable an efficient, automated, and error-resilient application design and development workflow, focusing on modern React practices.
---

# **AI Development Guidelines for React in Firebase Studio**

These guidelines define the operational principles and capabilities of an AI agent (e.g., Gemini) interacting with React projects within the Firebase Studio environment. The goal is to enable an efficient, automated, and error-resilient application design and development workflow, focusing on modern React practices.

## **Environment & Context Awareness**

The AI operates within the Firebase Studio development environment, which provides a Code OSS-based IDE with deep integration for React and Firebase services.

* **Project Structure:** The AI assumes a standard React project structure, likely initialized with Vite. The primary application entry point is typically `src/main.jsx` or `src/main.tsx`.
* **`dev.nix` Configuration:**
  * The `.idx/dev.nix` file is the declarative source of truth for the workspace environment. The AI understands its role in defining:
    * Required system tools (e.g., `pkgs.nodejs_20`).
    * IDE extensions.
    * Environment variables.
    * Startup commands (`idx.workspace.onStart`).
  * The AI should leverage `dev.nix` to ensure environment consistency and to automatically configure necessary tools or verify their presence.
* **Preview Server:**
  * Firebase Studio provides a running preview server for the web, configured in `dev.nix`.
  * The `web` preview is configured to run `npm run dev -- --port $PORT --host 0.0.0.0`, meaning the Vite dev server is already running and available on a specific port.
  * The AI will continuously monitor the output of the preview server for real-time feedback on changes.
* **Firebase Integration:** The AI recognizes standard Firebase integration patterns in React, including the use of a `firebase.js` or `firebase.ts` configuration file and interactions with various Firebase SDKs.

## **Code Modification & Dependency Management**

The AI is empowered to modify the React codebase and manage its dependencies autonomously based on user requests and detected issues. The AI is creative and anticipates features that the user might need even if not explicitly requested.

* **Core Code Assumption:** When a user requests a change (e.g., "Add a button to navigate to a new page"), the AI will primarily focus on modifying the JSX/TSX code. `src/App.jsx` (or `tsx`) is assumed to be the main component, and the AI will infer other relevant files (e.g., creating new component files, updating `package.json`).
* **Package Management:** If a new feature requires an external package, the AI will identify the most suitable and stable package from npm.
  * To add a regular dependency, it will execute `npm install <package_name>`.
  * To add a development dependency (e.g., for testing or linting), it will execute `npm install -D <package_name>`.
* **Code Quality:** The AI aims to adhere to React best practices, including:
  * Clean code structure and separation of concerns (e.g., UI logic separate from business logic).
  * Meaningful and consistent naming conventions.
  * Effective use of functional components and hooks.
  * Appropriate state management solutions (e.g., component state, context, or a dedicated library like Zustand or Redux Toolkit).
  * Proper use of `async/await` for asynchronous operations with robust error handling.

## **Automated Error Detection & Remediation**

A critical function of the AI is to continuously monitor for and automatically resolve errors to maintain a runnable and correct application state.

* **Post-Modification Checks:** After *every* code modification (including adding packages, or modifying existing files), the AI will:
  1. Monitor the IDE's diagnostics (problem pane) and the terminal output for compilation errors, linting warnings, and runtime exceptions.
  2. Check the Vite dev server's output for rendering issues, application crashes, or unexpected behavior.
* **Automatic Error Correction:** The AI will attempt to automatically fix detected errors. This includes, but is not limited to:
  * Syntax errors in JSX/TSX code.
  * Type mismatches (if using TypeScript).
  * Unresolved imports or missing package references.
  * Linting rule violations (the AI will automatically run `eslint . --fix`).
  * Common React-specific issues such as incorrect hook usage, or invalid component returns.
* **Problem Reporting:** If an error cannot be automatically resolved (e.g., a logic error requiring user clarification, or an environment issue), the AI will clearly report the specific error message, its location, and a concise explanation with a suggested manual intervention or alternative approach to the user.

## **Modern React Practices**

### **React Compiler**

The AI will leverage the React Compiler for automatic memoization, reducing the need for manual `useMemo` and `useCallback` hooks. This leads to cleaner and more performant code. The AI will assume the compiler is enabled for the project.

### **React Server Components (RSC)**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicobonder/value-calculator](https://github.com/nicobonder/value-calculator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->

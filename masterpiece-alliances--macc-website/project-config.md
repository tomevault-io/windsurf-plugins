---
trigger: always_on
description: This rule provides comprehensive best practices for developing React applications with Mobx, covering code organization, performance, testing, and security considerations. It aims to guide developers in building robust and maintainable applications using React-Mobx.
---

# React-Mobx Best Practices

This document outlines best practices for developing React applications with Mobx. Following these guidelines will help you build robust, maintainable, and performant applications.

## 1. Core Principles

- **Model observable state:**  Define the source of truth in your Mobx stores and make them observable.
- **Use derived state (computed properties):**  Derive data from your observable state using `computed` properties to avoid unnecessary re-renders and ensure data consistency.
- **Embrace mutability:** Mobx encourages direct mutation of observable state within actions.
- **Use strict mode:** Enforce predictable state changes by using `use strict` or enabling strict mode in Mobx to ensure all state modifications happen within actions.
- **Keep actions in MobX stores:** Encapsulate state mutations within Mobx actions to ensure data integrity and simplify debugging.
- **Prefer class vs object syntax:** Use class syntax for creating stores as it offers better structure, inheritance, and maintainability.
- **Inject store rather than importing:**  Use dependency injection (e.g., React Context) to provide stores to components instead of importing them directly to improve testability and reduce tight coupling.

## 2. Code Organization and Structure

- **Directory Structure:**
    - `/src`:
        - `/components`: React components (presentational and container components).
        - `/stores`: Mobx stores containing observable state and actions.
        - `/models`: Data models and types definitions.
        - `/services`: API clients and other services.
        - `/utils`: Utility functions.
        - `/hooks`: Custom React hooks.
        - `/constants`: Application-wide constants.
    - `/test`: Unit, integration, and end-to-end tests.

- **File Naming Conventions:**
    - Components: `ComponentName.jsx` or `ComponentName.tsx`
    - Stores: `StoreName.store.js` or `StoreName.store.ts`
    - Models: `ModelName.model.js` or `ModelName.model.ts`
    - Services: `ServiceName.service.js` or `ServiceName.service.ts`
    - Hooks: `useHookName.js` or `useHookName.ts`

- **Module Organization:**
    - Group related components, stores, and models into modules.
    - Use clear and descriptive module names.
    - Avoid circular dependencies between modules.

- **Component Architecture:**
    - **Presentational Components:**  Responsible for rendering UI and receiving data via props.
    - **Container Components:**  Connect presentational components to Mobx stores and manage state updates.
    - Use the `observer` decorator/function from `mobx-react-lite` to make components react to observable changes.

- **Code Splitting Strategies:**
    - Use React's `lazy` and `Suspense` components for lazy loading routes or components.
    - Implement route-based code splitting to load only necessary code for the current route.
    - Consider using `dynamic imports` for on-demand loading of modules.

## 3. Common Patterns and Anti-patterns

- **Design Patterns:**
    - **Observer Pattern:** Mobx uses the observer pattern to automatically update components when observable state changes.
    - **Dependency Injection:** Use React Context to inject stores into components, making them more testable and reusable.
    - **Provider Pattern:** Use a Provider component to make stores available to the component tree.

- **Recommended Approaches:**
    - **Form Handling:** Use Mobx to manage form state and validation.
    - **Asynchronous Operations:** Use Mobx actions to handle asynchronous operations and update the state accordingly.
    - **List Rendering:** Optimize list rendering using `key` props and `React.memo`.

- **Anti-patterns and Code Smells:**
    - **Modifying State Outside Actions:** Always modify observable state within Mobx actions to ensure data consistency.
    - **Deeply Nested Observables:** Avoid deeply nested observable objects as they can impact performance.
    - **Computing Data in Components:**  Use `computed` properties in stores to derive data instead of computing it directly in components.
    - **Over-Observing:** Only observe the specific parts of the store that a component needs.

- **State Management Best Practices:**
    - **Single Source of Truth:** Keep the application state in Mobx stores and avoid duplicating data.
    - **Normalization:** Normalize data to reduce redundancy and improve data consistency.
    - **Immutability (with Mutation):**  While Mobx encourages mutation, treat observable state as immutable from the perspective of components (i.e., components shouldn't directly mutate the store).

- **Error Handling Patterns:**
    - Implement centralized error handling using a dedicated error store.
    - Use try-catch blocks within actions to handle potential errors.
    - Display user-friendly error messages to the user.

## 4. Performance Considerations

- **Optimization Techniques:**
    - **`React.memo`:**  Wrap components with `React.memo` to prevent unnecessary re-renders when props haven't changed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masterpiece-alliances) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

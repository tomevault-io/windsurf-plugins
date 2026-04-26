---
trigger: always_on
description: description: This rule provides comprehensive best practices for developing with Shadcn UI, covering code organization, performance, security, and testing.
---

---
description: This rule provides comprehensive best practices for developing with Shadcn UI, covering code organization, performance, security, and testing.
globs: **/*.{js,jsx,ts,tsx}
---
# Shadcn UI Best Practices

This document outlines best practices for developing with Shadcn UI, covering code organization, common patterns, performance considerations, security, testing, common pitfalls, and tooling.

## 1. Code Organization and Structure

- **Directory Structure:**
    - Organize components into logical directories based on functionality or domain. For example, place form-related components in a `components/forms` directory.
    - Separate components into their own files.  Each component should have a dedicated file named after the component (e.g., `Button.tsx`).
    - Consider using an `index.ts` file within each directory to export all components from that directory, simplifying imports.
    - Structure directories to reflect the UI hierarchy.  For example, `/components/layout` for layout related components and `/components/ui` for reusable UI elements.

- **File Naming Conventions:**
    - Use PascalCase for component file names (e.g., `MyComponent.tsx`).
    - Use camelCase for variable and function names (e.g., `handleClick`).
    - Use descriptive names that clearly indicate the purpose of the component or function.

- **Module Organization:**
    - Break down complex components into smaller, reusable modules.
    - Keep components focused on a single responsibility.
    - Utilize shared utility functions and constants to avoid code duplication.  Create a `utils` directory for helper functions.
    - Use `components` directory to store UI components.

- **Component Architecture:**
    - Favor composition over inheritance. Create flexible components that can be customized through props.
    - Design components with clear separation of concerns: presentational components (UI) and container components (logic).
    - Use functional components with hooks for managing state and side effects.

- **Code Splitting Strategies:**
    - Implement lazy loading for non-critical components to improve initial load time.
    - Utilize React.lazy and Suspense for code splitting at the component level.
    - Configure your bundler (e.g., Webpack, Parcel) to automatically split code into smaller chunks.
    - Consider route-based code splitting for larger applications.

## 2. Common Patterns and Anti-patterns

- **Design Patterns Specific to Shadcn UI:**
    - Leverage the existing components provided by Shadcn UI whenever possible.
    - Customize components using styling solutions like Tailwind CSS's utility classes or CSS variables.
    - Create compound components by combining existing Shadcn UI components to build more complex UI elements.

- **Recommended Approaches for Common Tasks:**
    - Use Shadcn UI's form components (e.g., `Input`, `Select`) for handling user input.
    - Implement accessible components by following ARIA guidelines and using appropriate HTML semantics.
    - Use the `cn` utility (classnames library) provided by Shadcn UI to manage CSS class names effectively.

- **Anti-patterns and Code Smells to Avoid:**
    - Directly modifying the Shadcn UI component code.
    - Overusing custom CSS, as Shadcn UI is built with Tailwind CSS.
    - Neglecting accessibility considerations.
    - Creating overly complex components with too many responsibilities.

- **State Management Best Practices:**
    - Use React's built-in `useState` hook for simple component-level state.
    - Consider using a state management library like Zustand, Redux, or Recoil for more complex application state.
    - Avoid mutating state directly; always use the setState function or a state management library's update methods.

- **Error Handling Patterns:**
    - Implement error boundaries to catch errors in components and prevent the entire application from crashing.
    - Use try-catch blocks to handle errors in asynchronous operations and API calls.
    - Provide informative error messages to users.
    - Log errors to a monitoring service for debugging and analysis.

## 3. Performance Considerations

- **Optimization Techniques:**
    - Minimize re-renders by using `React.memo` for functional components and `shouldComponentUpdate` for class components.
    - Optimize event handlers by using useCallback to prevent unnecessary re-creation of functions.
    - Debounce or throttle expensive operations to reduce the frequency of execution.

- **Memory Management:**
    - Avoid memory leaks by properly cleaning up event listeners and timers in the `useEffect` hook.
    - Release unused resources, such as large data structures, when they are no longer needed.

- **Rendering Optimization:**
    - Use virtualized lists or grids for rendering large datasets.
    - Batch DOM updates to minimize reflows and repaints.
    - Use CSS containment to isolate rendering changes to specific parts of the DOM.

- **Bundle Size Optimization:**
    - Remove unused code and dependencies using tree shaking.
    - Minify JavaScript and CSS files to reduce their size.
    - Compress images using tools like ImageOptim or TinyPNG.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cici) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->

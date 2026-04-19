---
trigger: always_on
description: This rule enforces best practices and coding standards for projects using the Ant Design (antd) UI library within React applications. It covers code organization, performance, security, testing, and common pitfalls to ensure maintainable and efficient applications.
---

# Ant Design (antd) Best Practices and Coding Standards

This document outlines the recommended best practices for developing React applications using the Ant Design (antd) UI library. Following these guidelines will lead to more maintainable, performant, and secure applications.

## 1. Code Organization and Structure

### 1.1. Directory Structure

-   **`src/components/`**: Contains reusable React components, including those utilizing antd components.
-   **`src/pages/`**: Contains components representing different application routes or pages.
-   **`src/layouts/`**: Contains layout components that provide a consistent structure across pages.
-   **`src/services/`**: Contains modules for interacting with APIs and handling data fetching.
-   **`src/utils/`**: Contains utility functions and helper modules.
-   **`src/styles/`**: Contains global styles, theme customizations, and component-specific stylesheets.
-   **`src/assets/`**: Contains static assets such as images, fonts, and icons.
-   **`src/context/`**: (Optional) If using context API, store all context definition files here.

Example:


my-app/
├── src/
│   ├── components/
│   │   ├── Button.jsx
│   │   ├── Input.jsx
│   │   └── ...
│   ├── pages/
│   │   ├── HomePage.jsx
│   │   ├── LoginPage.jsx
│   │   └── ...
│   ├── layouts/
│   │   ├── MainLayout.jsx
│   │   └── ...
│   ├── services/
│   │   ├── api.js
│   │   └── ...
│   ├── utils/
│   │   ├── date-formatter.js
│   │   └── ...
│   ├── styles/
│   │   ├── global.css
│   │   ├── theme.js
│   │   └── ...
│   ├── App.jsx
│   └── index.js
└── ...


### 1.2. File Naming Conventions

-   **Components**: Use PascalCase for component file names (e.g., `MyComponent.jsx`, `UserProfile.tsx`).
-   **Styles**: Use kebab-case for style file names (e.g., `my-component.css`, `user-profile.module.scss`).
-   **Modules**: Use camelCase for module file names (e.g., `api.js`, `dateFormatter.ts`).

### 1.3. Module Organization

-   Group related components, styles, and assets within the same directory.
-   Create separate modules for API interactions, data transformations, and utility functions.

### 1.4. Component Architecture

-   **Presentational Components**: Focus on rendering UI elements and receiving data via props.
-   **Container Components**: Handle data fetching, state management, and logic, passing data to presentational components.
-   Use functional components with hooks whenever possible for simplicity and reusability.

### 1.5. Code Splitting

-   Utilize React.lazy and Suspense to load components on demand, improving initial load time.
-   Split routes into separate chunks to minimize the initial bundle size.
-   Consider using dynamic imports for less frequently used components or modules.

Example:

jsx
import React, { Suspense } from 'react';

const MyComponent = React.lazy(() => import('./MyComponent'));

function MyPage() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <MyComponent />
    </Suspense>
  );
}


## 2. Common Patterns and Anti-patterns

### 2.1. Design Patterns

-   **Higher-Order Components (HOCs)**: Use for cross-cutting concerns like authentication or data fetching.  Prefer hooks where possible.
-   **Render Props**:  An alternative to HOCs for sharing code between components.  Prefer hooks where possible.
-   **Compound Components**: Create reusable components with implicit state sharing (e.g., `Tabs` and `Tab` components).

### 2.2. Recommended Approaches

-   **Form Handling**: Use `antd`'s `Form` component for managing form state, validation, and submission.
-   **Data Display**: Leverage `antd`'s `Table`, `List`, and `Card` components for structured data presentation.
-   **Navigation**: Use `antd`'s `Menu` and `Breadcrumb` components for creating intuitive navigation.

### 2.3. Anti-patterns and Code Smells

-   **Direct DOM Manipulation**: Avoid directly manipulating the DOM; let React manage updates.
-   **Over-reliance on `any` type**: Using `any` in TypeScript defeats the purpose of static typing.  Provide explicit types.
-   **Mutating Props**: Treat props as read-only and avoid modifying them directly.
-   **Inline Styles**:  Keep styles in CSS files or use styled-components for better organization and maintainability. Prefer CSS Modules or Styled Components for component specific styles.

### 2.4. State Management

-   **Component State**: Use `useState` for simple, local component state.
-   **Context API**: Use for sharing state across multiple components without prop drilling.
-   **Redux/Zustand**: Consider for complex applications with global state and predictable state transitions.
-   **MobX**: Consider for complex applications where you want to observe changes in your data and derive calculations from that data.

### 2.5. Error Handling

-   **Try-Catch Blocks**: Use for handling synchronous errors.
-   **Error Boundaries**: Use to catch errors during rendering and prevent the entire application from crashing.
-   **Global Error Handling**: Implement a global error handler to log errors and provide user feedback.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frankvane) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->

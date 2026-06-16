---
trigger: always_on
description: description: This rule provides best practices and coding standards for Vue 3 projects, covering code organization, performance, security, testing, tooling, and common pitfalls to ensure maintainable and efficient applications. It aims to guide developers in writing high-quality Vue 3 code.
---

---
description: This rule provides best practices and coding standards for Vue 3 projects, covering code organization, performance, security, testing, tooling, and common pitfalls to ensure maintainable and efficient applications. It aims to guide developers in writing high-quality Vue 3 code.
globs: **/*.vue
---
- **Code Organization and Structure**:
  - **Directory Structure**: Adopt a feature-based directory structure. Group related files (components, stores, utilities) within feature-specific directories rather than separating by file type. This enhances maintainability and discoverability.
    - Example:
      
      src/
        components/
          MyComponent.vue
          ...
        views/
          MyView.vue
          ...
        features/
          user-profile/
            components/
              UserProfileCard.vue
            composables/
              useUserProfileData.js
            store/
              userProfile.js
          ...
      
  - **File Naming Conventions**: Use PascalCase for component file names (e.g., `MyComponent.vue`). Use camelCase for variable and function names (e.g., `myVariable`, `myFunction`). Use kebab-case for component selectors in templates (e.g., `<my-component>`).
  - **Module Organization**: Utilize ES modules (`import`/`export`) for modularity and code reusability. Group related functions and components into modules.
  - **Component Architecture**: Favor a component-based architecture. Design components to be small, reusable, and composable. Use props for data input and events for data output. Consider using a component library (e.g., Vuetify, Element Plus) for pre-built components.
  - **Code Splitting Strategies**: Implement lazy loading for components and routes to reduce initial bundle size. Use dynamic imports for on-demand loading of modules.
    - Example:
      javascript
      // Route-based code splitting
      const routes = [
        {
          path: '/about',
          component: () => import('./views/About.vue')
        }
      ]
      

- **Common Patterns and Anti-patterns**:
  - **Design Patterns**: Apply common design patterns such as composition API, provider/inject, and observer pattern where applicable.
    - **Composition API**: Organize component logic into composable functions for reusability and maintainability.
    - **Provider/Inject**: Use `provide` and `inject` to share data between components without prop drilling.
  - **Recommended Approaches**: Utilize `v-model` for two-way data binding, computed properties for derived state, and watchers for side effects. Use the Composition API for enhanced code organization and reusability.
  - **Anti-patterns and Code Smells**: Avoid directly mutating props. Avoid excessive use of global variables. Avoid complex logic within templates. Avoid tight coupling between components. Avoid over-engineering solutions.
  - **State Management**: Choose a state management solution (e.g., Vuex, Pinia) for complex applications.  Favor Pinia for Vue 3 due to its simpler API and improved TypeScript support. Decouple components from state management logic using actions and mutations.
  - **Error Handling**: Implement global error handling using `app.config.errorHandler`. Use `try...catch` blocks for handling synchronous errors. Utilize `Promise.catch` for handling asynchronous errors. Provide user-friendly error messages.
    - Example:
      javascript
      // Global error handler
      app.config.errorHandler = (err, vm, info) => {
        console.error('Global error:', err, info);
        // Report error to server or display user-friendly message
      }
      

- **Performance Considerations**:
  - **Optimization Techniques**: Use `v-once` for static content. Use `v-memo` to memoize parts of the template. Use `key` attribute for `v-for` loops to improve rendering performance.
  - **Memory Management**: Avoid creating memory leaks by properly cleaning up event listeners and timers. Use `onBeforeUnmount` lifecycle hook to release resources.
  - **Rendering Optimization**: Use virtual DOM efficiently. Minimize unnecessary re-renders by using `ref` and `reactive` appropriately. Use `shouldUpdate` hook in functional components to control updates.
  - **Bundle Size Optimization**: Use code splitting, tree shaking, and minification to reduce bundle size. Remove unused dependencies. Use smaller alternative libraries where possible.
  - **Lazy Loading**: Implement lazy loading for images, components, and routes. Use `IntersectionObserver` API for lazy loading images.

- **Security Best Practices**:
  - **Common Vulnerabilities**: Prevent Cross-Site Scripting (XSS) attacks by sanitizing user input. Prevent Cross-Site Request Forgery (CSRF) attacks by using CSRF tokens. Prevent SQL injection attacks by using parameterized queries.
  - **Input Validation**: Validate user input on both client-side and server-side. Use appropriate data types and formats. Escape special characters.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NA2CO333/elec-online](https://github.com/NA2CO333/elec-online) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->

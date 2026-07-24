---
trigger: always_on
description: You are working with the CoreUI Free Vue Admin Template, a professional admin dashboard built with Vue 3, CoreUI Vue components, and modern build tools. This project uses Vite for development and building, Vue Router for navigation, Pinia for state management, and Sass for styling.
---

# CoreUI Free Vue Admin Template - AI Assistant Rules

You are working with the CoreUI Free Vue Admin Template, a professional admin dashboard built with Vue 3, CoreUI Vue components, and modern build tools. This project uses Vite for development and building, Vue Router for navigation, Pinia for state management, and Sass for styling.

## Critical Rules

**Component Library**: ALWAYS use CoreUI Vue components from https://coreui.io/vue/docs/. NEVER use Tailwind CSS, Vuetify, Element Plus, or other component libraries. This project is built on Bootstrap 5 and CoreUI Vue components exclusively.

**Technology Stack**: This project uses:
- Vue 3.5.x with Composition API and script setup
- Single File Components (SFC) with <template>, <script>, and <style> sections
- CoreUI Vue 5.x and @coreui/coreui 5.x
- Vue Router 5.x for client-side routing
- Pinia 3.x for state management
- Vite 8.x for development server and building
- Sass/SCSS for styling with Bootstrap 5 variables
- Chart.js 4.x with @coreui/vue-chartjs for data visualization

## Code Conventions

**Vue/JavaScript Standards**:
- Use Composition API with `<script setup>` syntax
- Use ref() and reactive() for reactive state
- Follow Vue 3 Composition API patterns
- Use Prettier formatting: no semicolons, single quotes, 2-space indentation
- Enforce ESLint rules with Vue and Prettier plugins
- Prefer const and arrow functions
- Use destructuring where appropriate

**File Organization**:
- `src/` - All source code
  - `components/` - Reusable UI components (AppHeader, AppSidebar, etc.)
  - `views/` - Page components organized by feature (dashboard, forms, charts, etc.)
  - `layouts/` - Layout wrapper components (DefaultLayout)
  - `assets/` - Static assets (images, brand logos)
  - `scss/` - Global styles and theme customization
  - `router/` - Router configuration
  - `stores/` - Pinia stores
  - `_nav.js` - Navigation/sidebar menu configuration
  - `App.vue` - Main application component
  - `main.js` - Application entry point

**Vue/SFC Practices**:
- Use `<script setup>` for components (preferred modern syntax)
- Use `defineProps()` and `defineEmits()` for component API
- Use computed() for derived state
- Use watch() or watchEffect() for side effects
- Keep components focused and single-responsibility
- Extract reusable logic into composables (useX pattern)
- Use Suspense for async components when needed

**CSS/Sass Practices**:
- Import global styles in main.js: `import './scss/style.scss'`
- Use Bootstrap utilities first before custom CSS
- Leverage CoreUI CSS custom properties for theming
- Support dark mode through CoreUI's color mode system
- File: `src/scss/style.scss` - main stylesheet importing CoreUI and Bootstrap
- File: `src/scss/_custom.scss` - custom style overrides
- Use scoped styles in components: `<style scoped>`
- Use SCSS variables from Bootstrap and CoreUI when possible

**Routing Conventions**:
- Use createWebHashHistory for client-side routing (GitHub Pages compatible)
- Define routes in router/index.js as array of objects
- Use dynamic imports for lazy loading route components
- Use exact path matching where needed
- Public routes (login, register, 404, 500) defined separately
- Protected routes handled in DefaultLayout

**State Management**:
- Use Pinia for global state (theme, sidebar visibility)
- Create stores in stores/ directory
- Use defineStore() with setup syntax
- Access stores with const store = useStore()
- Keep component-level state in ref() or reactive() when state is local

**Naming Conventions**:
- PascalCase for component files and component names (AppHeader.vue, DefaultLayout.vue)
- camelCase for variables, functions, and composables (useState, useEffect)
- UPPER_SNAKE_CASE for constants (API_URL, MAX_ITEMS)
- kebab-case for CSS classes (following Bootstrap/CoreUI conventions)
- Descriptive names that indicate purpose (AppHeaderDropdown vs Dropdown)

## Project Structure

```
coreui-free-vue-admin-template/
├── public/              # Static assets served directly
├── src/
│   ├── assets/         # Images, logos, icons
│   │   ├── brand/     # Logo components
│   │   └── images/    # Image files
│   ├── components/     # Reusable UI components
│   │   ├── AppHeader.vue
│   │   ├── AppSidebar.vue
│   │   ├── AppFooter.vue
│   │   ├── AppContent.vue
│   │   ├── AppBreadcrumb.vue
│   │   └── AppHeaderDropdown.vue
│   ├── layouts/        # Layout components
│   │   └── DefaultLayout.vue
│   ├── views/          # Page components
│   │   ├── dashboard/ # Dashboard page
│   │   ├── base/      # Base UI components examples
│   │   ├── buttons/   # Button examples
│   │   ├── forms/     # Form examples
│   │   ├── charts/    # Chart examples
│   │   ├── icons/     # Icon examples
│   │   ├── notifications/ # Notification examples
│   │   ├── widgets/   # Widget examples
│   │   └── pages/     # Auth & error pages
│   ├── router/         # Router configuration
│   │   └── index.js   # Route definitions
│   ├── stores/         # Pinia stores
│   ├── scss/           # Stylesheets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coreui/coreui-free-vue-admin-template](https://github.com/coreui/coreui-free-vue-admin-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
